# Xamarin Dev Days ハンズオン手順書

これは、Xamarin.Forms と MSのクラウドサービス Microsoft Azure を使った簡単なアプリを作るハンズオンです。

Microsoft 本社の Xamarin チームが作った、詳細なハンズオン手順書『[Xamrin Dev Days Hands On Lab](https://github.com/xamarin/dev-days-labs/tree/2016/HandsOnLab)』の日本語訳版をここに載せます。


## 今回 何を作るの？

| 項目                       | 値                                                            |
|----------------------------|---------------------------------------------------------------|---|
| どんなアプリ？             | Xamarin Dev Days の本社スピーカーと、その人の詳細を表示するアプリ |
| どんなアプリ？(技術的視点) | Microsoft Azure に接続された Xamarin.Forms で作るアプリ       |

## 開発環境

Windows でも Mac でも良いです。

|OS|OS のバージョン|要インストール済|
|----|----|----|
|Windows|Windows 10|Visual Studio 2015 Update 3|
|Mac OS X|10.11 ("El Capitan") 以降 |Xamarin Studio, 最新の Xcode |

ことわり：    
この手順書はスクリーンショット盛り盛りでお送りしますが、Visual Studio でのスクリーンショットのみとさせていただきます。 Xamarin Studio でもだいたい似た感じなので大丈夫です。

# さぁ手を動かそう！

まず、[ハンズオンのレポジトリ](https://github.com/xamarin/dev-days-labs/tree/2016/HandsOnLab) を開いておいてください。    
[https://github.com/xamarin/dev-days-labs/tree/2016/HandsOnLab](https://github.com/xamarin/dev-days-labs/tree/2016/HandsOnLab)

## 手順 1 ：ソリューションファイルを開く

[Start ディレクトリ](https://github.com/xamarin/dev-days-labs/tree/2016/HandsOnLab/Start) の中にある「`DevDaysSpeakers.sln`」を開いてください。  
（Windows の場合は Visual Studio、Mac OS の場合は Xamarin Studio で開きます。）

ソリューションタブを見ると、4つのプロジェクトで構成されているのが分かります。

|#|プロジェクト名|概要|実行環境|
|----|----|----|----|
|1 | `DevDaysSpeakers (Portable)`|共通コード部分 (model とか view とか view model とか) が全部入った PCL (Portable Class Library )。|（不問）|
|2 | `DevDaysSpeakers.Droid`|Android アプリケーション|（不問）|
|3 | `DevDaysSpeakers.iOS`|iOS アプリケーション|実行には Mac が必要|
|4 | `DevDaysSpeakers.UWP`|Windows 10 UWP アプリケーション|実行には Windows 10 & VS 2015 が必要|

![DevDaysSpeakers.sln](https://camo.githubusercontent.com/29d9c125b34e962b95034db26e7195a3f53d82b0/687474703a2f2f636f6e74656e742e73637265656e636173742e636f6d2f75736572732f4a616d65734d6f6e74656d61676e6f2f666f6c646572732f4a696e672f6d656469612f34346634636161392d656662392d343430352d393564342d3733343136303865316330612f506f727461626c652e706e67)

共通部分である `DevDaysSpeakers (Portable)` プロジェクトの中に、空白の XAMLページ ([View/DetailsPage.xaml](https://github.com/xamarin/dev-days-labs/blob/master/HandsOnLab/Start/DevDaysSpeakers/DevDaysSpeakers/View/DetailsPage.xaml)など) がありますが、    
これはこのハンズオンの中で使うことになるものです。

## 手順 2 : NuGet Restore

すべてのプロジェクトに於いて、必要な NuGet パッケージはすべてインストール済みとなっています。なので、このハンズオンでは、新たにパッケージを追加でインストールする必要はありません。    

まず最初に我々がしなければならないことは、インターネットから、すべての NuGet パッケージをリストアすることです。

どうやるかというと、ソリューションタブの中の『ソリューション』を
右クリックして、『`Restore NuGet packages`』をクリックします。

![Restore NuGet packages](https://camo.githubusercontent.com/45c5469cf5ce35c1bf6b7dc2d235351874359bcc/687474703a2f2f636f6e74656e742e73637265656e636173742e636f6d2f75736572732f4a616d65734d6f6e74656d61676e6f2f666f6c646572732f4a696e672f6d656469612f61333161366266662d623435642d346336302d613630322d3133353966393834653830622f323031362d30372d31315f313332382e706e67)


## 手順 3 : Model を いじる

スピーカーの情報を取ってこれるようにするため、 Speaker モデルを作りましょう。

1. `DevDaysSpeakers (Portable)` プロジェクトの中の `DevDaysSpeakers/Model/Speaker.cs` ファイルを開きます。
1. 【コピペ】 Speaker クラスの中に、以下のプロパティを追加してください。

```csharp
public string Id { get; set; }
public string Name { get; set; }
public string Description { get; set; }
public string Website { get; set; }
public string Title { get; set; }
public string Avatar { get; set; }
```

## 手順 4 : View Model を いじる

Xamarin.Forms の view で、どのようにデータを表示するかのすべての機能を、`SpeakersViewModel.cs` が提供します。    
`SpeakersViewModel` は、 Speaker の List と、サーバからスピーカーのデータを取ってくるために呼ばれるメソッド、から構成されています。また、バックグラウンドタスクとしてデータを取って来ようとしていることを示す boolean フラグも持っています。   

### INotifyPropertyChanged をインプリメントしよう

`INotifyPropertyChanged` (= プロパティ値が変更されたことをクライアントに通知するインターフェイス) は、 MVVM フレームワークに於いて、重要なデータバインディングです。 それ（INotifyPropertyChanged）は、オブジェクトの状態に変更があった場合に、ユーザーインターフェースが通知を受け取ることができる仕組みを提供します。

今こうなっています：

[Before]
```csharp
public class SpeakersViewModel
{

}
```

これを、こうしてください：

[After]【コピペ】
```csharp
public class SpeakersViewModel : INotifyPropertyChanged
{

}
```

そして    
右クリック → 『`Implement Interface`』をクリック    
で、以下のコードが生えます。

```csharp
public event PropertyChangedEventHandler PropertyChanged;
```

これは、プロパティが変更された時いつでも呼ばれるメソッドです。ということで `OnPropertyChanged` というヘルパーメソッド(helper method)を作る必要が出てきます。

【コピペ】 C# 6
```csharp
void OnPropertyChanged([CallerMemberName] string name = null) =>
    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(name));
```

(ちなみに、C# 5 (バージョン古い)で書くとこうなります)
```csharp
void OnPropertyChanged([CallerMemberName] string name = null)
{
    var changed = PropertyChanged;

    if (changed == null)
        return;

    changed.Invoke(this, new PropertyChangedEventArgs(name));
}
```

というわけで、我々は、プロパティが更新され時はいつでも `OnPropertyChanged()` を呼ぶことができるようになりました。    
さぁ、我々の最初の、バインディングできるプロパティ (our first bindable property) を作ってみましょう！

### IsBusy プロパティ

この `SpeakersViewModel` クラスの中に、bool値を get/set するための、バッキングフィールド(backing field)と 自動プロパティ(自動実装プロパティ/auto-properties)を作りましょう。

まず、バッキングフィールドを作ります。

```csharp
bool busy;
```

そして、自動プロパティを作ります。

```csharp
public bool IsBusy
{
    get { return busy; }
    set
    {
        busy = value;
        OnPropertyChanged();
    }
}
```

`OnPropertyChanged();` を呼んでいますね。これを呼ぶことによって Xamarin.Forms は、IsBusy の値が set された時に、自動的に知ることができます。

### Speaker の ObservableCollection 

[メモ]    
`ObservableCollection` (自分の中身が変わったことを検知する仕組みを持っているコレクション)     

[TODO] We will use an ObservableCollection that will be cleared and then loaded with speakers.
なぜ `ObservableCollection` を使うかというと、これは要素(コレクションの中身)を追加とか削除とかすると発火する `CollectionChanged` というイベントを最初から持っているからです。

[TODO] In the class above the constructor simply create an autoproperty:

```csharp
public ObservableCollection<Speaker> Speakers { get; set; }
```

コンストラクタの中で、`ObservableCollection` の新しいインスタンスを作ります。
なので、SpeakersViewModelクラスのコンストラクタはこのようになります：

```csharp
public SpeakersViewModel()
{
    Speakers = new ObservableCollection<Speaker>();
}
```


### GetSpeakers メソッド

我々は今から `GetSpeakers` という名前のメソッドを作ろうとしていますが、これは、インターネットから speaker のデータをすべて取ってくるときに呼ぶメソッドです。    
まずは単純な HTTP リクエストから実装します。でもあとから Azure からデータを取ってきたりこちら(クライアント)から変更して sync できるようにアップデートします！

まず、`GetSpeakers`という名前のメソッドを作ります。型は `async Task` です。

```csharp
async Task GetSpeakers()
{

}
```

これから、この `GetSpeakers` メソッドの中に色々とコードを書いていきます。

まず、既にデータを取得済みかをチェックします。

```csharp
async Task GetSpeakers()
{
    if(IsBusy)
        return;
}
```
次に、 try/catch/finally ブロックのための土台をいくつか作ります。

```csharp
async Task GetSpeakers()
{
    if (IsBusy)
        return;

    Exception error = null;
    try
    {
        IsBusy = true;

    }
    catch (Exception ex)
    {
        error = ex;
    }
    finally
    {
       IsBusy = false;
    }
}
```

`IsBusy` は最初は true にセットしておき、そしてサーバーに接続しようとした時と、終わった時は false にセットします。

そして、`try`ブロックの中で、サーバから json データを取ってくるために `HttpClient` を使います。

```csharp
using(var client = new HttpClient())
{
    // サーバから json データを取ってくる
    var json = await client.GetStringAsync("http://demo4404797.mockable.io/speakers");
} 
```

## Details
それでは詳細画面とその画面遷移を行なっていきましょう． **SpeakersPage.xaml** のコードビハインドである **SpeakersPage.xaml.cs** を開いてください．

### ItemSelected event
コードビハインドである **SpeakersPage.xaml.cs** 内で SpeakersViewModelのセットアップをしていることが見つかると思います．
それではスピーカーのアイテムが選択された時に，選択されたという通知を受け取るためイベントを追加してみましょう．
コード内の **BindingContext = vm;** の下に以下のように **ListViewSpeakers** にイベントを追加していきます．

```csharp
ListViewSpeakers.ItemSelected += ListViewSpeakers_ItemSelected;
```

また選択されたアイテムの詳細ページのであるDetailsPageに遷移するために，以下のようなメソッドを実装します．

```csharp
private async void ListViewSpeakers_ItemSelected(object sender, SelectedItemChangedEventArgs e)
{
    var speaker = e.SelectedItem as Speaker;
    if (speaker == null)
        return;

    await Navigation.PushAsync(new DetailsPage(speaker));

    ListViewSpeakers.SelectedItem = null;
}
```

上記のコードでは，選択されたアイテムがnullでないことを確認し，
そのアイテムのDetailsPageをもともと組み込まれている **Navigation** APIを使用し新しいページとしてプッシュし，
その後にListViewで選択されたアイテムの選択状態を解除しています．

### DetailsPage.xaml
では、Details pageに記入してみましょう。SpeakersPageと同様です。私たちはStackLayoutを使用しますが、でも私たちはScrollViewでそれをラップします。私たちが長いテキストを持っている場合、

```xml
  <ScrollView Padding="10">
    <StackLayout Spacing="10">
     <!-- Detail controls here -->
    </StackLayout>    
  </ScrollView>
```

では、Speaker objectのプロパティのためのコントロールとバインディングを追加してみましょう。

```xml
<Image Source="{Binding Avatar}" HeightRequest="200" WidthRequest="200"/>
      
<Label Text="{Binding Name}" FontSize="24"/>
<Label Text="{Binding Title}" TextColor="Purple"/>
<Label Text="{Binding Description}"/>
```

今、楽しみのため、 let's add two buttons that we will add click events to in the code behind:

```xml
<Button Text="Speak" x:Name="ButtonSpeak"/>
<Button Text="Go to Website" x:Name="ButtonWebsite"/>
```

### Text to Speech
もし、私たちが **DetailsPage.xaml.cs** をオープンアップする場合、私たちはハンドラを多少クリックするだけで、すぐに追加できます。ButtonSpeakで始めましょう。その場所で、私たちは[Text To Speech Plugin](https://github.com/jamesmontemagno/TextToSpeechPlugin)を使ってスピーカーの説明を読み戻すことになるでしょう。

コンストラクタでは、 BindingContext以下にクリックハンドラを追加します。 

```csharp
ButtonSpeak.Clicked += ButtonSpeak_Clicked;
```

それから私たちはクリックハンドラを追加できて、また、クロスプラットフォームAPIをコールでき、テキストをスピーチします。

```csharp
private void ButtonSpeak_Clicked(object sender, EventArgs e)
{
    CrossTextToSpeech.Current.Speak(this.speaker.Description);
}
```

### Open Website
Xamarin.Forms それ自身が持っているものとしては、クロスプラットフォーム目的で書かれてビルトされたナイスなAPIsを持っています。どのような物かというと、デフォルトのブラウザでURLを開くようなものです。

その他のクリックハンドラを追加しましょう。 でも今回はButtonWebsiteのためです。

```csharp
ButtonWebsite.Clicked += ButtonWebsite_Clicked;
```

それから私たちは、デバイスキーワード (Device keyword) を使用できます。OpenUri method をコールします。

```csharp
private void ButtonWebsite_Clicked(object sender, EventArgs e)
{
    if (speaker.Website.StartsWith("http"))
        Device.OpenUri(new Uri(speaker.Website));
}
```

### Compile & Run
いま私たちがすべきは、コンパイルのためのセットをすべて行う必要があり、そして、まさに前のようにrunする必要があります。
