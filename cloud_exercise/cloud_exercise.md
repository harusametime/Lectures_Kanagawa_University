## 演習の目的

本コンテンツは神奈川大学 情報学部 情報リテラシー演習受講生向けに作られたページです。

本講義は AWS の学習環境 [AWS Academy](https://aws.amazon.com/jp/training/awsacademy/) を使ってクラウドの基本を演習を通して学ぶことを目的としています。情報学部の学生の多くが、ウェブサイトの作成と公開を初期段階で学ぶことから、それに沿った形で、AWS Academy の追加コンテンツとして独自に作成し
ています。 コンテンツに関する問い合わせは AWS Academy ではなく github の issue にて報告をお願いします。

## 演習の概要

クラウドオブジェクトストレージであるAmazon S3 に HTML ファイル (CSS ファイルもあれば含める) をアップロードして、ウェブサイトを公開します。
また、エッジサービスである Amazon CloudFront 経由でウェブサイトにアクセスします。

注意事項:
今回使用する環境は長期間利用できる環境ではないため、演習が完了したらスクリーンショットをとって結果を残しておいてください。
スクリーンショットはレポートの提出などで利用してください。


## AWS Academy のアカウント登録

学生が AWS Academy を利用するためにはアカウントを登録する必要があります。教員が受講生をコースに追加すると「コースへの招待」というメールが届くため、それに従って学生はアカウントを登録します。

<img width=400 src="../images/account_creation.png">


## AWS Academy へのログイン

アカウント登録が終わったら、以下のページから AWS Academy へログインします。Student Login を選んでください。
https://www.awsacademy.com/vforcesite/LMS_Login

<img width=400 src="../images/login_page.png"> 

## コースの選択とモジュールの選択

左のメニューからコースを選択し、AWS Academy Cloud Foundations を選択します。

<img width=600 src="../images/select_course.png" border =1>

  
次にモジュールから 「サンドボックス ラボ」を選択します。

<img width=600 src="../images/select_module.png"  border =1 >


## AWS 環境の起動

初回は同意事項に関するページが表示されますので、一番下までスクロールして I Agree をクリックし、同意してください。

<img width=600 src="../images/agreement.png" border =1>

するとサンドボックスの説明画面が開きます。右上のメニューから Start Lab ボタンをクリックすると、Labをスタートすることができます。ボタンを押すと環境構築中の別画面が開きますが、1-2分たったら閉じて問題ありません。AWS ボタンを押すとAWS コンソールの画面を開くことができます。
もしAWSボタンを押してもAWSコンソール画面が開かない場合は、まだ環境構築中の可能性があるので、もう少し待ってみます。

<img width=600 src="../images/start_lab.png">

## AWS コンソールの操作

### Amazon S3 

Amazon S3 は AWS のオブジェクトストレージのサービスで様々なデータを保管できるサービスです。
AWS コンソールが起動したら左上の検索ウィンドウに S3 と入力して、Amazon S3 のサービスを検索します。

<img width=600 src="../images/search_service.png" border =1>

まず最初にバケットの作成を行います。これまでの利用状況に応じて、トップ画面に「バケットを作成」ボタンがあったり、左のメニューの「バケット」からバケット一覧にアクセスして、「バケットを作成」ボタンを確認できたりします。

<img width=600 src="../images/create_bucket.png " border =1>

バケット作成で最初の画面では、デフォルトのまま汎用バケットを選択し、バケット名を入力します。バケット名は世界で唯一の名前である必要があります。

<img width=600 src="../images/naming_bucket.png " border =1>

次にバケットのパブリックアクセスのブロック設定を行います。Amazon S3のバケットはデフォルトで公開しない（公開しようとしてもブロックする）設定になっています。これは意図せず公開してしまい、情報が漏洩すること防ぐためです。
今回はWebページを公開する目的で利用するので、このブロック設定を解除します。あくまで、ブロックしないようにするだけなので、この段階で情報が公開されるわけではありません。**パブリックアクセスをすべてブロックをオフ**にして、**最後の確認のところにチェック**をいれてください。

<img width=600 src="../images/public_bucket.png " border =1>

最後に作成ボタンを押すとバケットを作成できます。バケットの一覧画面に戻ると作成したバケットを確認できるので、バケットの名前をクリックして詳細を確認します。

<img width=600 src="../images/check_bucket.png " border =1>

開いた画面の上部にタブが並んでいるので「アクセス許可」を選びます。ここでバケットを公開する設定を行います。

<img width=600 src="../images/bucket_policy.png " border =1>

バケットを公開するかどうかは、そのバケットの取扱の詳細を決める**バケットポリシー**を設定することによって行います。したがって、上部画面から**バケットポリシー**の**編集**ボタンをクリックして、バケットポリシーを作成します。

編集画面を開くと何も書かれていない空のエディタが開くので、下の**バケットポリシーのテンプレート**をまずコピーします。その後、Resource の "(バケットARN)" のところに、編集画面の上部にあるバケット ARN をコピーして貼り付け、その後に `/*` を追加してください。Resource の内容は例えば以下のようになります。

```json
"Resource": "arn:aws:s3:::test-samejima/*"
```

このバケットポリシーの意味は、だれでも (Principal: *)、リソース (Resource": "バケットARN") からのダウンロード操作 ("Action": "s3:GetObject") を許可 ("Effect": "Allow") するという意味です。

終わったらバケットポリシーを保存してください。

<img width=700 src="../images/edit_bucket_policy.png " border =1>

**■ バケットポリシーのテンプレート**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
        "Sid": "PublicReadForGetBucketObjects", 
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "(バケットARN)/*"
        }
    ]
}
```

公開したいファイル (HTML, CSSなど)をアップロードします。上のタブの**オブジェクト**から**アップロード**ボタンをクリックするとファイルアップロード画面になります。ここに HTML, CSS, 画像 などをアップロードします。**ファイル名は英語にしてください**。

<img width=700 src="../images/upload_files.png " border =1>

最後に静的ホスティングを有効にしてアクセスできるかどうか確認します。まず上部のタブから **プロパティ**を選択し、一番下の**静的ウェブサイトホスティング**の**編集**をクリックします。インデックスのドキュメントは最初に表示したいHTMLファイルを記載します。


<img width=700 src="../images/static_hosting.png " border =1>

ではアクセスしてみましょう。**プロパティ**から一番下の**静的ウェブサイトホスティング**の項目をチェックするとURLが掲載されています。URLをクリックして、先程指定したHTMLファイルが表示されたら成功です。もしバケットにフォルダ （例えば `website`) を作成して、そこに html ファイルを置く場合は、URL最後に `/website/`を追加してアクセスします (例えば、http://(バケット名).s3-website-us-east-1.amazonaws.com/website/)

<img width=700 src="../images/check_hosting.png " border =1>

このような感じで表示されましたか？

<img width=500 src="../images/web_example.png " border =1>



### Amazon CloudFront

Amazon CloudFrontは、AWSが提供するコンテンツ配信ネットワーク（CDN）サービスです。ウェブサイトやアプリケーションのコンテンツを、世界中に分散されたエッジロケーションを通じて高速に配信することで、ユーザーのアクセス速度を向上させることができます。

まず左上の検索ウィンドウから CloudFront を検索します。

<img width=500 src="../images/cloudfront.png " border =1>

CloudFront の画面を開いたら、CloudFront ディストリビューションを作成します。

<img width=500 src="../images/create_dist.png" border =1>

ディストリビューションの名前を設定して Nextをクリックします。

<img width=500 src="../images/config_dist.png" border =1>

次にオリジン、つまり CloudFront が参照するもとのファイルの場所を設定します。さきほどファイルを置いた**Amazon S3**を選び、S3 Origin は S3 で設定したバケットの場所を指定します（Browse S3 から選択できます)。もし`website`のようなフォルダ以下に html ファイルを入れた場合は、**Original path** に `website` といれてください。

<img width=500 src="../images/set_origin.png" border =1>

Enable Security のところは、**セキュリティ保護を有効にしないでください**を選択して、Next をクリックします。

<img width=500 src="../images/set_security.png" border =1>

最後にこれまでの確認しましょう。確認して間違いなければ Create distribution をクリックします。

<img width=500 src="../images/confirm_config_dist.png" border =1>

次にアップロードしたファイルにルートからアクセスする場合はルートオブジェクトの設定が必要です。
ディストリビューション一覧にもどると、作成したディストリビューションを確認できるので、ディストリビューション名をクリックします。**一般**タブを開いていることを確認し、**設定**から**編集**を選びます。Settings の画面下部の Default root object に、最初に表示すべきファイル名を入れます。

<img width=500 src="../images/set_root_object.png" border =1>

最後に確認しましょう。**一般**タブを開くと、ディストリビューションドメイン名があるので、これをコピーしてブラウザに貼り付けます。Amazon S3 で開いたウェブサイトと同じものが表示されれば成功です。実際に CloudFront で表示されるまでに少し時間がかかることがあります。

<img width=500 src="../images/complete_and_check.png" border =1>
