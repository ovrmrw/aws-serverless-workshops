# Module 5: Enabling 3rd party applications using OAuth 2.0

このモジュールでは、Wild Rydes アプリケーションをプラットフォームに変え、サードパーティの開発者が私たちのAPIの上に新しいアプリケーションを構築できるようにします。 サードパーティの開発者と協力することで、新しい市場や地域を開拓したり、ライドに新しい機能を提供したりすることが容易になります。

> In this module we will turn our Wild Rydes application into a platform, enabling third party developers to build new applications on top of our APIs. Working with third party developers makes it easier for us to open new markets and geographies as well as provide new functionality for our riders.

TODO: ↓の訳があやしいので直す。

モジュール2で構築した Cognito ユーザープールの OAuth 2.0 フローを有効にします。 OAuth を使用して、サードパーティの開発者はあなたのAPIの上に新しいクライアントアプリケーションを構築することができます。 アプリケーションのAPIに新しいメソッドを作成し、ユニコーンがそれらのライドを一覧表示できるようにします。 これは私達にとって新たな事業ラインを開き、第三者の開発者がユニコーンが彼らの時間と収益を管理するのを助けるアプリケーションを構築することを容易にするでしょう。 まず、ライドを一覧表示するための新しいメソッドを作成します。 次に、Cognito ユーザープールで OAuth フローを有効にしてサンプルクライアントをデプロイします。

> You'll configure your Cognito User Pool from module #2 to enable OAuth 2.0 flows. Using OAuth, third party developers can build new client applications on top of your APIs. We will create a new method in the application's API that allows unicorns to list the rides they have given. This will open a new line of business for us, making it easy for third party developers to build applications that help unicorns manage their time and earnings. First, we will create the new method to list rides. Then, we will enable OAuth flows in our Cognito User Pool and deploy a sample client.

![OAuth 2.0 3rd party app architecture](../images/oauth-architecture.png)

上の図は、新しいサードパーティアプリケーションのコンポーネントが現在の Wild Rydes アーキテクチャとどのように対話するかを示しています。 Webアプリケーションは S3 バケットにデプロイされています。 アプリケーションは、Cognito ユーザープールの組み込みUIを使用して、暗黙の認可 OAuth 2.0 フローを開始し、ユーザーを認証します。 ユーザーが認証されると、クライアントアプリケーションは Unicorn のIDとアクセストークンを受け取ります。 Unicorn のトークンには、新しいAPIへのアクセスを許可する追加の「Unicorn」クレームが含まれています。 API Gateway では、カスタムオーソライザーが、Cognito によって生成されたJWTアクセストークン内の「Unicorn」クレームをチェックし、ユニコーン名をバックエンドの Lambda 関数に渡します。 バックエンドの Lambda 関数は、アクセストークンからのユニコーン名を使用して DynamoDB の `Rides` テーブルをクエリします。

> The diagram above shows how the component of the new third party application interact with our current Wild Rydes architecture. The web application is deployed in an S3 bucket. The application uses the Cognito User Pools built-in UI to start an implicit grant OAuth 2.0 flow and authenticate the user. Once the Unicorn user is authenticated, the client application receives an identity and access token for the Unicorn. Tokens for Unicorns include an additional `Unicorn` claim that gives them access to the new API. In API Gateway, a custom authorizer checks for the `Unicorn` claim in the JWT access token produced by Cognito and passes the unicorn name to the backend Lambda function. The backend Lambda function uses the unicorn name from the access token to query the rides table in DynamoDB.

### Prerequisites

このモジュールは、Wild Rydes ワークショップのこれまでの4つのモジュールすべてに依存しています。 使いやすくするために、完全なスタックを起動できる CloudFormation テンプレートを用意しました。 以前のモジュールをスキップして CloudFormation テンプレートを使用してデプロイした場合は、aws-serverless-workshop リポジトリをローカルの作業環境に複製してください。

**社内ワークショップメモ: 共有アカウントでは CloudFormation は使用できません。**

> This module depends on all of the previous four modules in the Wild Rydes workshop. To make it easier to get started, we have prepared a CloudFormation template that can launch the complete stack for you. If you have skipped the earlier modules, and deploying using the CloudFormation template, clone the aws-serverless-workshop repository to your local working environment.

アカウントにモジュール1から4までのリソースを以前に作成していて、それでも以下の CloudFormation テンプレートを使用して新たに始めたい場合は、最初に [cleanup steps](../9_CleanUp/) に従ってください。

> If you have previously created resources from modules #1 to #4 in your account, and would still like to start fresh with the CloudFormation template below, make sure you first follow the [cleanup steps](../9_CleanUp/).

Region| Launch
------|-----
US East (N. Virginia) | [![Launch Modules 1, 2, 3, and 4 in us-east-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=wildrydes-webapp&templateURL=https://s3.amazonaws.com/wildrydes-us-east-1/WebApplication/5_OAuth/prerequisites.yaml)
US East (Ohio) | [![Launch Modules 1, 2, 3, and 4 in us-east-2](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks/new?stackName=wildrydes-webapp&templateURL=https://s3.amazonaws.com/wildrydes-us-east-2/WebApplication/5_OAuth/prerequisites.yaml)
US West (Oregon) | [![Launch Modules 1, 2, 3, and 4 in us-west-2](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=wildrydes-webapp&templateURL=https://s3.amazonaws.com/wildrydes-us-west-2/WebApplication/5_OAuth/prerequisites.yaml)
EU (Frankfurt) | [![Launch Modules 1, 2, 3, and 4 in eu-central-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/new?stackName=wildrydes-webapp&templateURL=https://s3.amazonaws.com/wildrydes-eu-central-1/WebApplication/5_OAuth/prerequisites.yaml)
EU (Ireland) | [![Launch Modules 1, 2, 3, and 4 in eu-west-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new?stackName=wildrydes-webapp&templateURL=https://s3.amazonaws.com/wildrydes-eu-west-1/WebApplication/5_OAuth/prerequisites.yaml)
EU (London) | [![Launch Modules 1, 2, 3, and 4 in eu-west-2](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=eu-west-2#/stacks/new?stackName=wildrydes-webapp&templateURL=https://s3.amazonaws.com/wildrydes-eu-west-2/WebApplication/5_OAuth/prerequisites.yaml)
Asia Pacific (Tokyo) | [![Launch Modules 1, 2, 3, and 4 in ap-northeast-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/new?stackName=wildrydes-webapp&templateURL=https://s3.amazonaws.com/wildrydes-ap-northeast-1/WebApplication/5_OAuth/prerequisites.yaml)
Asia Pacific (Seoul) | [![Launch Modules 1, 2, 3, and 4 in ap-northeast-2](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-northeast-2#/stacks/new?stackName=wildrydes-webapp&templateURL=https://s3.amazonaws.com/wildrydes-ap-northeast-2/WebApplication/5_OAuth/prerequisites.yaml)
Asia Pacific (Sydney) | [![Launch Modules 1, 2, 3, and 4 in ap-southeast-2](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-2#/stacks/new?stackName=wildrydes-webapp&templateURL=https://s3.amazonaws.com/wildrydes-ap-southeast-2/WebApplication/5_OAuth/prerequisites.yaml)
Asia Pacific (Mumbai) | [![Launch Modules 1, 2, 3, and 4 in ap-south-1](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-south-1#/stacks/new?stackName=wildrydes-webapp&templateURL=https://s3.amazonaws.com/wildrydes-ap-south-1/WebApplication/5_OAuth/prerequisites.yaml)

スタック作成プロセスで **Website Bucket Name** を要求されます。**wildrydes-webapp-&lt;username&gt;** など、バケットに一意の名前を指定してください。

> The stack creation process will ask you for a **Website Bucket Name**, specify a unique name for your bucket such as **wildrydes-webapp-&lt;username&gt;**.

#### Populate the rides database

スタックが正常に作成されたら、CloudFormationコンソールの **Outputs** タブを開きます。 **WebsiteURL** の値をコピーしてブラウザウィンドウでページに移動します。

> After the stack created successfully, open the **Outputs** tab in the CloudFormation console. Copy the **WebsiteURL** output value and navigate to the page with a browser window.

Wild RydesのWebサイトで、**Giddy Up！** ボタンをクリックして新しいユーザーを登録します。 確認コードを受け取ったら、Webサイトの **verify.html** ページに移動してコードを送信してください。 ログインページから、新しい認証情報を使用してWebサイトにログインします。 アプリケーションを使用して、いくつかのユニコーンライドをリクエストします。このモジュールの後半には、ライドデータが必要になります。

> On the Wild Rydes website, click the **Giddy Up!** button and register a new user. Once you have received your verification code, navigate to the **verify.html** page of the website to submit your code. From the login page, use your new credentials to log into the website. Use the application to request a few unicorn rides, we will need the rides data later in this module.


### 1. Create the new List Rides Lambda function

#### Background

AWS Lambda は、APIリクエストに応じてコードを実行します。 このステップでは、リストライドAPIに対するユニコーンリクエストに答えるための新しい Lambda 関数を作成します。 Wild Rydes アプリケーションでは、各APIメソッドを独立した Lambda 関数にマッピングしました。 単一の Lambda 関数に複数のAPIメソッドをグループ化することもできます。 馴染みのあるライブラリでコードを書き続けるために、2つのオープンソースフレームワークを作成しました。[aws-serverless-express](https://github.com/awslabs/aws-serverless-express) と [aws-serverless-java-container](http://github.com/awslabs/aws-serverless-java-container) です。

> AWS Lambda runs your code in response to an API request. In this step, you will create a new Lambda functions to answer unicorn requests to the list rides API. In the Wild Rydes application, we have mapped each API method to an independent Lambda function. You also have the option to group multiple API methods in a single Lambda function. To keep writing code with the libraries you are already familiar with, we have created two open source frameworks: [aws-serverless-express](https://github.com/awslabs/aws-serverless-express) and [aws-serverless-java-container](http://github.com/awslabs/aws-serverless-java-container).

[listUnicornRides.js](./listUnicornRides.js) ファイルのコードを見てください。 Lambda 関数は、現在のユニコーン名がイベントのオーソライザーコンテキストに存在することを想定しています。 イベントが解析されると、関数は現在のユニコーンのすべての行を抽出するために DynamoDB の `Rides` テーブルをクエリします。 このフィールドは、次のステップで作成するカスタムオーソライザーによって設定されます。

> Take a look at the code in the [listUnicornRides.js](./listUnicornRides.js) file. The Lambda function expects the current unicorn name to be present in the authorizer context of the event. Once the event is parsed, the function queries our DynamoDB rides table to extract all of the rows for the current unicorn. The field is set by the custom authorizer you'll create in the next step.

#### High-Level Instructions

AWS Lambda コンソールを使用して、APIリクエストを処理する **ListUnicornRides** という新しい Lambda 関数を作成します。 提供されている [listUnicornRides.js](./listUnicornRides.js?raw=1) サンプル実装を関数コードに使用してください。 そのファイルから AWS Lambda コンソールのエディターにコピーして貼り付けるだけです。

> Use the AWS Lambda console to create a new Lambda function called **ListUnicornRides** that will process the API requests. Use the provided [listUnicornRides.js](./listUnicornRides.js?raw=1) example implementation for your function code. Just copy and paste from that file into the AWS Lambda console's editor.

このワークショップのモジュール2で作成した `WildRydesLambda` IAM ロールを使用するように関数を必ず設定してください。

> Make sure to configure your function to use the `WildRydesLambda` IAM role you created in module 2 of this workshop.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. **サービス** のコンピューティングセクションから **Lambda** を選択します。

1. **関数の作成** を選択します。

1. **一から作成** カードを選択します。

1. **関数名** に `ListUnicornRides-{Your Name}` と入力します。

1. **ランタイム** として **Node.js 8.10** を選択します。

1. **既存のロール** ドロップダウンで前のモジュールで作成した `WildRydesLambda-{Your Name}` を選択します。

1. **関数の作成** をクリックします。

1. [listUnicornRides.js](./listUnicornRides.js?raw=1) の内容を関数コードエディタに貼り付けます。

1. (**社内ワークショップメモ: 共用アカウントの場合は50行目の `TableName: 'Rides',` の `Rides` を `Rides-{Your Name}` に変更する必要があります。**)

1. ハンドラフィールドが **index.handler** になっていることを確認します。

1. 画面右上にある **保存** をクリックします。

(Original)

1. Choose on **Services** then select **Lambda** in the Compute section.

1. Choose **Create function**.

1. Click the **Author from scratch** button at the top of the blueprint list.

1. Enter **ListUnicornRides** in the **Name** field.

1. Select **wildrydes/WildRydesLambda** from the **Existing Role** dropdown.

	 ![Define handler and role screenshot](../images/lambda-handler-and-role.png)

1. Click **Create function**.

1. Select **Node.js 6.10** for the **Runtime**.

1. Copy and paste the code from [listUnicornRides.js](./listUnicornRides.js?raw=1) into the code entry area.

    ![Create Lambda function screenshot](../images/create-list-rides-function.png)

1. Leave the default of **index.handler** for the **Handler** field.

1. Click **Save** at the top of the page.

</p></details>

### 2. Create the new custom authorizer Lambda function

#### Background

Amazon API Gateway は AWS Lambda 関数を利用して認証を決定できます。 JWTトークンなどのベアラトークンをサポートするために、カスタムオーソライザーを使用できます。 カスタム認証機能を使用して設定されている場合、API Gateway はリクエストトークンとコンテキストを使用して Lambda 関数を呼び出します。 Lambda カスタム認証プログラムは、呼び出された特定のメソッドだけでなく、API Gateway がAPI全体の認証決定を行うために使用できるポリシーを返す必要があります。 カスタムオーソライザーの作成を容易にするために、Lambda コンソールから選択できるJavaScriptとPythonのブループリントを作成しました。 これらのブループリントには、ポリシー生成を簡素化するユーティリティオブジェクトが含まれています。

> Amazon API Gateway can leverage an AWS Lambda function to make authorization decisions. In order to support bearer tokens, such as JWT tokens, you can use custom authorizers. When configured with a custom authorizer, API Gateway invokes a Lambda function with the request token and context. The Lambda custom authorizer must return a policy that API Gateway can use to make the authorization decision for the entire API, not just the specific method that was called. To make the creation of custom authorizers easier, we have created JavaScript and Python blueprints that you can select from the Lambda console. These blueprints contain a utility object that simplifies policy generation.

リクエストコンテキスト値に追加されたキーと値のペアのセットを返すこともできます。 私たちのカスタムオーソライザーのためのコードは `ListUnicornAuthorizer` フォルダーにあります、そのフォルダーを開いて、そして私たちのカスタムオーソライザーがどのように動作するかのアイデアを得るために `index.js` ファイルを見てください。 私たちの新しいリストライドAPIへのアクセスを許可するために `UnicornManager/unicorn` と呼ばれるカスタムスコープに頼ります - このスコープは Unicorn マネージャアプリケーションによって生成されたクライアントトークンに自動的に追加されます。

> You can also return a set of key/value pairs that are appended to the request context values. The code for our custom authorizer is in the `ListUnicornAuthorizer` folder, open the folder and take a look at the `index.js` file to get an idea of how our custom authorizer works. To authorize access to our new list rides API we rely on a custom scope called `UnicornManager/unicorn` - this scope is automatically added to client tokens produced by the Unicorn Manager application.

#### High-Level Instructions

AWS Lambda コンソールを使用して、インカミングJWTベアラトークンを処理する **ListUnicornAuthorizer** という新しいLambda 関数を作成します。 提供された [ListUnicornAuthorizer.zip](./ListUnicornAuthorizer.zip) を関数コードとしてアップロードします。 オーソライザー Lambda 関数は **`USER_POOL_ID`** という名前の環境変数に依存し、これを Lambda コンソールで定義し、Cognito コンソールから WildRydes **Pool Id** の値を設定します。

> Use the AWS Lambda console to create a new Lambda function called **ListUnicornAuthorizer** that will process incoming JWT bearer tokens. Upload the provided [ListUnicornAuthorizer.zip](./ListUnicornAuthorizer.zip) as the function code. The authorizer Lambda function relies on an environment variable called **`USER_POOL_ID`**, define this in the Lambda console and set the value of the WildRydes **Pool Id** from the Cognito console.

このワークショップのモジュール2で作成した **WildRydesLambda** IAM ロールを使用するように関数を必ず設定してください。

> Make sure to configure your function to use the **WildRydesLambda** IAM role you created in module 2 of this workshop.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. **サービス** のコンピューティングセクションから **Lambda** を選択します。

1. **関数の作成** を選択します。

1. **一から作成** カードが選択されていることを確認します。

1. **関数名** に `ListUnicornAuthorizer-{Your Name}` と入力します。

1. **ランタイム** として **Node.js 8.10** を選択します。

1. **既存のロール** ドロップダウンで前のモジュールで作成した `WildRydesLambda-{Your Name}` を選択します。

1. **関数の作成** をクリックします。

1. **コード エントリ タイプ** を **.zip ファイルをアップロード** に変更します。

1. ハンドラフィールドが **index.handler** になっていることを確認します。

1. **アップロード** ボタンをクリックし、ローカルにダウンロードした [ListUnicornAuthorizer.zip](./ListUnicornAuthorizer.zip) を選択します。

1. **環境変数** セクションにキー名 **USER_POOL_ID**、値 **(Cognito ユーザープールのID)** のペアを入力します。 これは前のモジュールで作成したIDです。

1. ページ上部にある **保存** をクリックします。

(Original)

1. Choose on **Services** then select **Lambda** in the Compute section.

1. Choose **Create function**.

1. Click the **Author from scratch** button at the top of the blueprint list.

1. Enter **ListUnicornAuthorizer** in the **Name** field.

1. Select **wildrydes/WildRydesLambda** from the **Existing Role** dropdown.

1. Click **Create function**.

1. Change the **Code entry type** to **Upload a .ZIP file**.

1. Select **Node.js 6.10** for the **Runtime**.

1. Leave the default of **index.handler** for the **Handler** field.

1. Click the **Upload** button and select the [ListUnicornAuthorizer.zip](./ListUnicornAuthorizer.zip) file in the current module folder.

1. Expand the **Environment variables** section and declare a new variable called **USER_POOL_ID**. The value for the variable is the **Pool Id** for the WildRydes user pool, you can find the value in the Cognito console.

    ![Create Lambda function screenshot](../images/create-list-rides-authorizer-function.png)

1. Click **Save** at the top of the page.

</p></details>

### 3. Configure the new custom authorizer

#### Background

Amazon API Gateway は AWS Lambda の機能を利用して認証を決定できます。 これにより、舞台裏のビジネスロジックをカスタマイズできます。 API Gateway は、2種類のカスタムオーソライザーをサポートしています。**Token authorizers** と **Request authorizers**。 認証の決定が純粋にクライアントのベアラトークンに基づいている場合は、トークンオーソライザーを使用できます。 リクエストのオーソライザーはあなたの Lambda 関数に、bodyを除くすべてのリクエスト情報へのアクセス権を与えます。

> Amazon API Gateway can leverage AWS Lambda functions to make authorization decision. This enables you to customize the business logic behind the scenes. API Gateway supports two type of custom authorizers: **Token authorizers** and **Request authorizers**. You can use Token authorizers when your authorization decision is purely based on the client's bearer token. Request authorizers give your Lambda function access to all of the request information except for the body.

API Gateway は、カスタムオーソライザーからコンテキスト情報を受け取り、それらをバックエンドサービスに渡すこともできます。 私たちのアプリケーションでは、 `UnicornManager` スコープがトークンに存在する場合、カスタム認証プログラムはリクエストコンテキストに `unicorn` プロパティを含みます [(参照)](./ListUnicornAuthorizer/index.js#L109)。

> API Gateway can also receive context information from the custom authorizer and pass them to the backend service. In our application, the custom authorizer includes the `unicorn` property in the request context if the `UnicornManager` scope [is present in the token](./ListUnicornAuthorizer/index.js#L109).

#### High-level Instructions

API Gateway コンソールを開き、モジュール4で作成した **WildRydes** APIで新しいオーソライザーを作成します。 オーソライザーは、前の手順で作成した **ListUnicornAuthorizer** 関数を使用する必要があります。 新しいオーソライザーを **Token authorizer** として設定し、トークンソースは **Authorization** ヘッダーにする必要があります。

> Open the API Gateway console and create a new authorizer in the **WildRydes** API we created in module #4. The authorizer should use the **ListUnicornAuthorizer** function we created in the previous step. You should configure the new authorizer as a **Token authorizer** and the token source should be the **Authorization** header.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. **サービス** から「ネットワーキング ＆ コンテンツ配信」の下の **API Gateway** を選択します。

1. **WildRydes-{Your Name}** カードを選択し、左のメニューから **オーソライザー** を選択します。

1. **+ 新しいオーソライザーの作成** ボタンをクリックします。

1. **名前** に `ListUnicornAuthorizer` と入力し、**タイプ** は **Lambda** を選択します。

1. **Lambda 関数** フィールドをクリックして `ListUnicornAuthorizer-{Your Name}` を選択します。

1. **Lambda 呼び出しロール** フィールドを空白のままにします。 このように設定されていると、API Gateway コンソールは、呼び出しを許可するために Lambda 関数に対する許可を自動的に設定します。 新しいオーソライザー設定を保存すると、コンソールからこの操作を確認するように求められます。

1. **Lambda イベントペイロード** は **トークン** を選択し、**トークンのソース** に `Authorization` と入力します。

1. **認証のキャッシュ** はデフォルト状態のままにして **作成** をクリックします。

1. API Gateway コンソールから、Lambda 関数に対する新しい権限を確認するように求められます。 **Grant & Create** をクリックします。

(Original)

1. Open the **Services** menu and select **API Gateway** in the Application Services section.

1. Open the **WildRydes** API in the left menu and select the **Authorizers** page.

    ![Open custom authorizers](../images/open-wild-rydes-authorizers.png)

1. Click the button to **Create New Authorizer** at the top of the page.

1. Enter **ListUnicornAuthorizer** as the **Name** and **Lambda** as the **Type**.

1. Using the **Lambda Function** field, select your region and enter the **ListUnicornAuthorizer** Lambda function name.

1. Leave the **Lambda Execution Role** field blank. Configured this way, the API Gateway console automatically sets the permissions on the Lambda function to allow the invocation. The console will ask you to confirm this action as you save the new authorizer settings.

1. Select **Token** as the **Lambda Event Payload** and enter **Authorization** as the **Token Source**.

1. Leave the default values in the **Authorization Caching** settings and click **Create**

    ![Create Custom Token Authorizer](../images/create-custom-token-authorizer.png)

1. The API Gateway console asks you to confirm the new permissions on the Lambda function. Click **Grant & Create**.

</p></details>

### 4. Create the new API Gateway method

#### Background

REST規約に従って、ライドを一覧表示するために `/ride` リソースの `GET` メソッドを使います。 同様に、特定のライドのデータを抽出したい場合は、 `/ride/{rideId}` という新しいリソースを作成し、このリソースの下で `GET` メソッドを使用して特定のライドのデータを抽出します。 [REST Resource Naming Guide](https://restfulapi.net/resource-naming/) を見てください。

> Following REST conventions, you will use a `GET` method on the `/ride` resource to list the rides. In the same fashion, if we wanted to extract data for a specific ride, we would create a new resource called `/ride/{rideId}` and use a `GET` method under this resource to extract the data for a specific ride. Take a look at the [REST Resource Naming Guide](https://restfulapi.net/resource-naming/).

#### High-Level Instructions

API Gateway コンソールで、モジュール4で作成した `WildRydes` APIを開き、`/ride` リソースに新しい **GET** メソッドを追加します。 メソッドの統合は、このモジュールのステップ1で作成した **ListUnicornRides** 関数への **Lambda プロキシ** 統合です。 前の手順で作成した **ListUnicornAuthorizer** を認証に使用するように新しいメソッドを構成します。 APIリソースに変更を加えたら、新しい設定を既存の **prod** ステージにデプロイします。

> In the API Gateway console, open the `WildRydes` API we created in module #4 and add a new **GET** method to the `/ride` resource. The method integration should be a **Lambda Proxy** integration to the **ListUnicornRides** function we created in step #1 of this module. Configure the new method to use the **ListUnicornAuthorizer** we created in the previous step for authorization. Once you have made the changes to the API resources, deploy the new configuration to the existing **prod** stage.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. Open the **サービス** menu and select **API Gateway** in the Application Services section.

1. Open the **WildRydes-{Your Name}** API and, from the **リソース** page, select the `/ride-{Your Name}` resource.

1. Using the **アクション** dropdown menu in the **リソース** pane, select **メソッドの作成**.

1. Configure the new method as a **GET** and confirm the settings with the small checkmark button next to the dropdown.

1. In the method integration settings screen, select **Lambda 関数** as the **統合タイプ**, check the **Lambda プロキシ統合の使用** checkbox, then select your Lambda region and use `ListUnicornRides-{Your Name}` (careful: NOT `ListUnicornAuthorizer`) as the function name.

1. Click **保存** and confirm the new permissions on the Lambda function by clicking **OK** in the modal window.

1. In the **メソッドの実行** screen, open the **メソッドリクエスト** pane.

1. Click on the pencil icon next to the **認証** settings to change the value and select the **ListUnicornAuthorizer** from the dropdown.

1. Click the checkmark icon next to the dropdown to save your changes.

1. Using the **アクション** dropdown in the **リソース** pane, select **API のデプロイ**.

1. In the deployment modal window, select the **prod** stage from the **デプロイされるステージ** dropdown and then click **デプロイ**.

(Original)

1. Open the **Services** menu and select **API Gateway** in the Application Services section.

1. Open the **WildRydes** API and, from the **Resources** page, select the `/ride` resource.

1. Using the **Actions** dropdown menu in the **Resources** pane, select **Create Method**.

1. Configure the new method as a **GET** and confirm the settings with the small checkmark button next to the dropdown.

1. In the method integration settings screen, select **Lambda Function** as the **Integration Type**, check the **Use Lambda Proxy Integration** checkbox, then select your Lambda region and use **ListUnicornRides** (careful: NOT **ListUnicornAuthorizer**) as the function name.

    ![Configure List Rides integration](../images/list-rides-api-integration.png)

1. Click **Save** and confirm the new permissions on the Lambda function by clicking **Ok** in the modal window.

1. In the **Method Execution** screen, open the **Method Request** pane.

1. Click on the pencil icon next to the **Authorization** settings to change the value and select the **ListUnicornAuthorizer** from the dropdown.

    ![Configure Custom Authorizer](../images/select-list-custom-authorizer.png)

1. Click the checkmark icon next to the dropdown to save your changes.

1. Using the **Actions** dropdown in the **Resources** pane, select **Deploy API**.

1. In the deployment modal window, select the **prod** stage from the **Deployment stage** dropdown and then click **Deploy**.

</p></details>

### 5. Create S3 bucket for static website

#### Background

Unicorn Manager と呼ばれる私たちの新しいパートナーのウェブサイトも、Amazon S3 でホストされている静的なアプリケーションです。 バケットポリシーを使用して、S3 バケットのコンテンツにアクセスできるユーザーを定義できます。 バケットポリシーは、どのプリンシパルがバケット内のオブジェクトに対してさまざまなアクションを実行できるかを指定するJSONドキュメントです。

> Our new partner website, called Unicorn Manager, is also a static application hosted on Amazon S3. You can define who can access the content in your S3 buckets using a bucket policy. Bucket policies are JSON documents that specify what principals are allowed to execute various actions against the objects in your bucket.

デフォルトでは、S3 バケット内のオブジェクトは、 `http://&lt;Regional-S3-prefix&gt;.amazonaws.com/<bucket-name>/<object-key>` という構造のURLを介して使用できます。 ルートURL（`/index.html` など）からアセットを配信するには、バケットでウェブサイトのホスティングを有効にする必要があります。 これにより、オブジェクトはバケット( `<bucket-name>.s3-website-<AWS-region>.amazonaws.com` )のAWSリージョン固有のWebサイトエンドポイントで利用可能になります。

> By default objects in an S3 bucket are available via URLs with the structure http://&lt;Regional-S3-prefix&gt;.amazonaws.com/<bucket-name>/<object-key>. In order to serve assets from the root URL (e.g. /index.html), you'll need to enable website hosting on the bucket. This will make your objects available at the AWS Region-specific website endpoint of the bucket: <bucket-name>.s3-website-<AWS-region>.amazonaws.com

私たちのアプリケーションはリダイレクトを必要とする OAuth 2.0 の暗黙の流れを介して Cognito と対話するので、私たちのウェブサイトではHTTPSを利用する必要があります。 S3静的Webサイト用のHTTPSエンドポイントを取得するには、 [CloudFront distribution](https://aws.amazon.com/cloudfront/) を使用できます。

> Because our application interacts with Cognito via the OAuth 2.0 implicit flow, which requires a redirect, we need our website to use HTTPS. To have an HTTPS endpoint for an S3 static website, we can use a [CloudFront distribution](https://aws.amazon.com/cloudfront/).

#### High-Level Instructions

コンソールまたは AWS CLI を使用して Amazon S3 バケットを作成します。 あなたのバケットの名前はすべてのリージョンとアカウントにわたってグローバルにユニークでなければならないことに留意してください。 `unicornmanager-firstname-lastname` のような名前を使うことを勧めます。 バケット名がすでに存在するというエラーが表示された場合は、未使用の名前が見つかるまで数字または文字を追加してみてください。

> Use the console or AWS CLI to create an Amazon S3 bucket. Keep in mind that your bucket's name must be globally unique across all regions and customers. We recommend using a name like `unicornmanager-firstname-lastname`. If you get an error that your bucket name already exists, try adding additional numbers or characters until you find an unused name.

匿名ユーザーにサイトを閲覧させるには、新しい Amazon S3 バケットにバケットポリシーを追加する必要があります。 デフォルトでは、あなたのバケットはあなたのAWSアカウントへのアクセス権を持つ認証されたユーザーによってのみアクセス可能になります。 [付与するポリシーの例](http://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-2) を参照してください。 匿名ユーザーへの読み取り専用アクセス このポリシー例では、インターネット上の誰でもあなたのコンテンツを閲覧することができます。 バケットポリシーを更新する最も簡単な方法は、コンソールを使用することです。 バケットを選択し、アクセス権限タブを選択してからバケットポリシーを選択します。

> You will need to add a bucket policy to your new Amazon S3 bucket to let anonymous users view your site. By default your bucket will only be accessible by authenticated users with access to your AWS account. See [this example](http://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-2) of a policy that will grant read only access to anonymous users. This example policy allows anyone on the Internet to view your content. The easiest way to update a bucket policy is to use the console. Select the bucket, choose the permission tab and then select Bucket Policy.

コンソールを使用して、静的Webサイトホスティングを有効にします。 バケットを選択した後、これをプロパティタブで実行できます。 インデックスドキュメントとして `index.html` を設定し、エラードキュメントを空白のままにします。 詳しくは [configuring a bucket for static website hosting](https://docs.aws.amazon.com/AmazonS3/latest/dev/HowDoIWebsiteConfiguration.html) に関するドキュメントを参照してください。

> Using the console, enable static website hosting. You can do this on the Properties tab after you've selected the bucket. Set `index.html` as the index document, and leave the error document blank. See the documentation on [configuring a bucket for static website hosting](https://docs.aws.amazon.com/AmazonS3/latest/dev/HowDoIWebsiteConfiguration.html) for more details.

CloudFront コンソールを使用して、S3静的WebサイトのURLをオリジンドメインおよび / をパスとして指定して、Webコンテンツ用の新しいディストリビューションを作成します。 配布がHTTPS要求のみを受け入れ、HTTP要求がHTTPSのURLにリダイレクトされることを確認してください。

> Using the CloudFront console, create a new Distribution for web content specifying the S3 static website URL as the origin domain and / as the path. Make sure that the distribution only accepts HTTPS requests and HTTP requests are redirected to the HTTPS url.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the AWS Management Console choose **サービス** then select **S3** under Storage.

1. Choose **+ バケットを作成する**

1. Provide a globally unique name for your bucket such as `unicornmanager-{Your Name}`.

1. Select the Region you've chosen to use for this workshop from the dropdown.

1. Choose **作成** in the lower left of the dialog without selecting a bucket to copy settings from.

1. Open the bucket you just created.

1. Choose the **アクセス権限** tab, then click the **バケットポリシー** button.

1. (**社内ワークショップメモ: バケットポリシー の更新作業をする前に、モジュール1で作業したときと同じように パブリックアクセス設定 を変更する必要があります。**)

1. Enter the following policy document into the bucket policy editor replacing `[YOUR_BUCKET_NAME]` with the name of the bucket you created in section 1:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::[YOUR_BUCKET_NAME]/*"
            }
        ]
    }
    ```

1. Choose **保存** to apply the new policy. You will see a warning indicating `このバケットにはパブリックアクセス権限があります`. This is expected.

1. Next, choose the **プロパティ** tab.

1. Choose the **Static website hosting** card.

1. Select **このバケットを使用してウェブサイトをホストする** and enter `index.html` for the Index document. Leave the other fields blank.

1. Note the **エンドポイント** URL at the top of the dialog before choosing **保存**.

1. Click **保存** to save your changes.

1. Next, open the **CloudFront** console under the **ネットワーキング ＆ コンテンツ配信**.

1. In the **CloudFront Distributions** page, click **Create Distribution**.

1. For the delivery method, under **Web** section, click **Get Started**.

1. In the **Origin Domain Name** field, paste the URL for the S3 static website we just created and **/** as the origin path. **Do not select the bucket from dropdown list, paste the full website url including the http:// prefix. The origin type should be `custom`, not `s3`**.

1. In the **Viewer Protocol Policy** make sure that **Redirect HTTP to HTTPS** is selected.

1. Under **Distribution Settings** for **Price Class**, select **Use Only US, Canada and Europe**.

1. Click **Create Distribution** at the bottom of the page.

1. Creating a global distribution can take some time. Let CloudFront do its work in the background and move on the next step. We will come back to get the distribution endpoint at a later step.

(Original)

1. In the AWS Management Console choose **Services** then select **S3** under Storage.

1. Choose **+ Create Bucket**

1. Provide a globally unique name for your bucket such as `unicornmanager-firstname-lastname`.

1. Select the Region you've chosen to use for this workshop from the dropdown.

1. Choose **Create** in the lower left of the dialog without selecting a bucket to copy settings from.

    ![Create bucket screenshot](../images/create-unicornmanager-bucket.png)

1. Open the bucket you just created.

1. Choose the **Permissions** tab, then click the **Bucket Policy** button.

1. Enter the following policy document into the bucket policy editor replacing `[YOUR_BUCKET_NAME]` with the name of the bucket you created in section 1:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::[YOUR_BUCKET_NAME]/*"
            }
        ]
    }
    ```

    ![Update bucket policy screenshot](../images/update-bucket-policy.png)

1. Choose **Save** to apply the new policy. You will see a warning indicating `This bucket has public access`. This is expected.

1. Next, choose the **Properties** tab.

1. Choose the **Static website hosting** card.

1. Select **Use this bucket to host a website** and enter `index.html` for the Index document. Leave the other fields blank.

1. Note the **Endpoint** URL at the top of the dialog before choosing **Save**.

1. Click **Save** to save your changes.

    ![Enable website hosting screenshot](../images/enable-website-hosting-unicornmanager.png)

1. Next, open the **CloudFront** console under the **Networking & Content Delivery**.

1. In the **CloudFront Distributions** page, click **Create Distribution**.

1. For the delivery method, under **Web** section, click **Get Started**.

1. In the **Origin Domain Name** field, paste the URL for the S3 static website we just created and **/** as the origin path. **Do not select the bucket from dropdown list, paste the full website url including the http:// prefix. The origin type should be `custom`, not `s3`**.

1. In the **Viewer Protocol Policy** make sure that **Redirect HTTP to HTTPS** is selected.

    ![Create CloudFront distribution](../images/create-cloudfront-distribution.png)

1. Under **Distribution Settings** for **Price Class**, select **Use Only US, Canada and Europe**.

1. Click **Create Distribution** at the bottom of the page.

1. Creating a global distribution can take some time. Let CloudFront do its work in the background and move on the next step. We will come back to get the distribution endpoint at a later step.

</p></details>

### 6. Declare a new client application

#### Background

Amazon Cognito ユーザープールを使用すると、プールと対話できる複数のクライアントアプリケーションを宣言できます。 これには、あなたが所有するアプリケーションとサードパーティの開発者によるアプリケーションの両方が含まれます。 各アプリケーションは、アプリケーションIDとクライアントシークレットによって識別されます。 Cognito ユーザープールは、登録、ログイン、パスワードのリセット、MFAなどの最も一般的なユーザー操作をサポートする、ホスト型ログインUIも提供します。 ホストされているUIの外観をカスタマイズすることもできます。

> Amazon Cognito User Pools allows you to declare multiple client applications that can interact with your pool. This includes both applications you own and apps by third party developers. Each application is identified by an application id and client secret. Cognito User Pools also offers a hosted login UI that supports the most common user operations such as registration, login, reset passwords, and MFA. You can also customize the look and feel of the hosted UI.

#### High-Level Instructions

Cognito コンソールを使用して、**UnicornManager** という新しいクライアントアプリケーションを追加します。 クライアントアプリケーションは S3 でホストされ、JavaScriptで書かれた静的なWebサイトなので、クライアントシークレットは必要ありません。 次に、Cognito コンソールの App Integration セクションで、ホステッドログインUIのドメイン名プレフィックスを設定します。 これを **WildRydes-&lt;username&gt;** と呼びます。

> Using the Cognito console, add a new client application called **UnicornManager**. Because the client application is a static website hosted on S3 and written in JavaScript, we do **not** need a client secret. Next, in the App Integration section of the Cognito console, configure a domain name prefix for your hosted login UI. We called this **WildRydes-&lt;username&gt;**.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the AWS Management Console choose **サービス** then select **Cognito** under Mobile.

1. In the intro page, click **ユーザープールの管理** an open the **WildRydes-{Your Name}** pool.

1. Open the **アプリクライアント** from the **全般設定** menu on the left.

1. Click **別のアプリクライアントの追加**.

1. Enter **UnicornManager-{Your Name}** as the **アプリクライアント名** and uncheck the **クライアントシークレットを生成** checkbox.

1. Click **アプリクライアントの作成**.

1. Open the **ドメイン名** configuration page.

1. Specify a unique custom domain name, for example **wildrydes-{Your Name}**.

1. Make sure that the domain name is available and then click **変更の保存**.

(Original)

1. In the AWS Management Console choose **Services** then select **Cognito** under Mobile.

1. In the intro page, click **Manage your User Pools** an open the **WildRydes** pool.

1. Open the **App clients** from the **General settings** menu on the left.

1. Click **Add another app client**.

1. Enter **UnicornManager** as the **App client name** and uncheck the **Generate client secret** checkbox.

    ![Create bucket screenshot](../images/create-cognito-app-client.png)

1. Click **Create app client**.

1. Open the **Domain name** configuration page.

1. Specify a unique custom domain name, for example **wildrydes-sapessi**.

1. Make sure that the domain name is available and then click **Save changes**.

</p></details>

### 7. Create the Unicorns scope in the Cognito User Pool

#### Background

Amazon Cognito ユーザープールを使用すると、カスタムリソースサーバーを宣言できます。 カスタムリソースサーバーは一意の識別子（通常はサーバーのURI）を持ち、カスタムスコープを宣言できます。 カスタムアプリケーションがユーザープールのスコープを要求することを許可できます。 ユーザーがこれらのアプリケーションで認証を受けると、Cognito がホストするUIがユーザーの認証とアクションの承認を行います。 カスタムクレームはJWTアクセストークンに自動的に追加されます。

> Amazon Cognito User Pools lets you declare custom resource servers. Custom resource servers have a unique identifier - normally the server uri - and can declare custom scopes. You can allow custom applications to request scopes in your user pools. When users authenticate with these applications, the Cognito hosted UI takes care of authenticating the user and authorizing the action. Custom claims are automatically added to the JWT access token.

#### High-Level Instructions

Cognito コンソールを使用して、**WildRydes** ユーザープールを開き、**UnicornServer** という新しいカスタムリソースサーバーを作成します。 **UnicornServer** は **Identifier** として **UnicornManager** を使用し、**unicorn** スコープを許可する必要があります。

> Using the Cognito console, open the **WildRydes** User Pool and create a new custom resource server called **UnicornServer**. The **UnicornServer** should use **UnicornManager** as the **Identifier** and allow the **unicorn** scope.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. Open the **サービス** menu and select **Cognito** in the Mobile section.

1. In the main screen, select **ユーザープールの管理**.

1. Open the **WildRydes-{Your Name}** pool and select **リソースサーバー** under **アプリの統合**.

1. In the resource servers screen, click **リソースサーバーの追加**.

1. Specify **UnicornServer-{Your Name}** as the **Name**.

1. Use **UnicornManager-{Your Name}** as the  **Identifier** for the custom resource server.

1. In the **スコープ** section, declare a new scope called **unicorn**. I've used "**Allow listing of rides for unicorns**" as the description.

1. Click **変更の保存** to create your new custom resource server.

(Original)

1. Open the **Services** menu and select **Cognito** in the Mobile section.

1. In the main screen, select **Manage your User Pools**.

1. Open the **WildRydes** pool and select **Resource Servers** under **App integration**.

    ![Open resource servers](../images/cognito-resource-servers-menu.png)

1. In the resource servers screen, click **Add a resource server**.

1. Specify **UnicornServer** as the **Name**.

1. Use **UnicornManager** as the  **Identifier** for the custom resource server.

1. In the **Scopes** section, declare a new scope called **unicorn**. I've used "**Allow listing of rides for unicorns**" as the description.

    ![Configure Cognito Resource Server](../images/configure-cognito-resource-server.png)

1. Click **Save changes** to create your new custom resource server.

</p></details>

### 8. Configure the new app client for OAuth

#### Background

Amazon Cognito ユーザープールは、認証コード付与、暗黙的付与、およびクライアント認証情報付与をサポートしています。 サードパーティの開発者は、Cognito がホストするUIにアプリケーションIDをロードして、有効になっている任意のフローを要求できます。 Cognito ユーザープールは、カスタム認証フローを構築するために使用できる一連のクライアントおよびサーバー/管理者APIも公開します。 認証が成功した結果、Cognito は OpenID Connect 互換のIDトークンとJWTアクセストークンを生成します。 アクセストークンには、アプリケーションに対して宣言したカスタムスコープが含まれています。

> Amazon Cognito User Pools supports the authorization code grant, implicit, and client credentials grants. Third party developers can load the Cognito hosted UI with their application ID and request any of the enabled flows. Cognito User Pools also exposes a set of client and server/admin APIs that you can use to build custom authentication flows. As a result of a successful authentication Cognito produces and OpenID Connect-compatible identity token and a JWT access token. The access token includes the custom scopes you declared for the application.

この例では、単純にするために暗黙のフローを使用します。 暗黙の許可フローは、主にモバイルアプリケーションによって使用されます。 Webアプリケーションの場合は、通常、サードパーティの開発者が独自のバックエンドサービスをホストし、認証コード付与フローを使用することを要求します。

> In our example, we will use the implicit flow for the sake of simplicity. Implicit grant flows are mostly used by mobile applications. For web applications, you would normally require third party developers to host their own backend service and use the authorization code grant flow.

#### High-Level Instructions

**アプリクライアントの設定** を開き、**Cognito ユーザープール** をIDプロバイダーとして使用し、**Implicit grant** フローを許可するように **UnicornManager** アプリケーションを設定します。 アプリケーションが手順7で作成した **カスタムスコープ** にアクセスできることを確認してください。 コールバックURLとして、手順5で作成した CloudFront ディストリビューションエンドポイントを使用します。 コールバックURLは `https：// xxxxxxxxxxx.cloudfront.net` のようになります。

> Open the **App client settings** and configure the **UnicornManager** app to use **Cognito User Pool** as an identity provider and allow the **Implicit grant** flow. Make sure the application has access to the **custom scope** we created in step #7. As a callback URL, use the CloudFront distribution endpoint we created in step #5. The callback url will look like this: `https://xxxxxxxxxxx.cloudfront.net`.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. In the AWS Management Console choose **サービス** then select **Cognito** under Mobile.

1. In the intro page, click **ユーザープールの管理** an open the **WildRydes-{Your Name}** pool.

1. Open the **アプリクライアントの設定** from the **アプリの統合** menu on the left. This page lists both the app clients declared for your user pool. Make sure you make the following changes only to the **UnicornManager-{Your Name}** client app.

1. Select **Cognito ユーザープール** as an identity provider for the app client.

1. Enable the **Implicit grant** OAuth flow and allow the **UnicornManager-{Your Name}/unicorn** custom scope.

1. In the **コールバック** and **サインアウト** URLs, specify the HTTPS CloudFront distribution endpoint adding **https://** at the beginning and **/** at the end:

    1. You can find the distribution endpoint in the **CloudFront** console.
    1. Select the distribution we created in step #5.
    1. In the **General** tab, copy the value for **Domain name**.

1. Click **変更の保存**.

(Original)

1. In the AWS Management Console choose **Services** then select **Cognito** under Mobile.

1. In the intro page, click **Manage your User Pools** an open the **WildRydes** pool.

1. Open the **App clients settings** from the **App integration** menu on the left. This page lists both the app clients declared for your user pool. Make sure you make the following changes only to the **UnicornManager** client app.

1. Select **Cognito User Pool** as an identity provider for the app client.

1. Enable the **Implicit grant** OAuth flow and allow the **UnicornManager/unicorn** custom scope.

1. In the **Callback** and **Signout** URLs, specify the HTTPS CloudFront distribution endpoint adding **https://** at the beginning and **/** at the end:
 1. You can find the distribution endpoint in the **CloudFront** console.
 1. Select the distribution we created in step #5.
 1. In the **General** tab, copy the value for **Domain name**.

    ![Create bucket screenshot](../images/configure-cognito-app-client.png)

1. Click **Save changes**.

</p></details>

### 9. Configure and upload the Unicorn Manager application to S3

#### Background

最後のステップは、新しい Cognito アプリケーションIDを使用してクライアントコードを設定し、S3 バケットにアップロードすることです。

> The last step is to configure the client code with the new Cognito application id and upload to our S3 bucket.

#### High-Level Instructions

**UnicornManager** フォルダにある `config.js` ファイルを開き、`userPoolClientId` を Cognito の新しい UnicornManager アプリケーションIDに置き換え、手順6で設定したリージョンとドメインプレフィックスを設定します。 最後に、**WildRydesApiInvokeUrl** 値を前提条件の CloudFormation スタック出力から設定ファイルの **invokeUrl** プロパティにコピーします。 ファイルを保存して閉じます。

> Open the `config.js` file in the **UnicornManager** folder, replace the `userPoolClientId` with the new UnicornManager application id from Cognito, set the region and the domain prefix we configured in step #6. Finally, copy the **WildRydesApiInvokeUrl** value from the prerequisites CloudFormation stack output into the **invokeUrl** property of the config file. Save and close the file.

**UnicornManager** フォルダの内容を S3 バケットのルートにアップロードします。 この手順を完了するには、AWSマネジメントコンソール（Google Chromeブラウザが必要）または AWS CLI を使用できます。 ローカルマシンに AWS CLI がすでにインストールおよび設定されている場合は、その方法を使用することをお勧めします。 それ以外の場合は、最新バージョンのGoogle Chromeがインストールされている場合はコンソールを使用してください。

> Upload the content of the **UnicornManager** folder to the root of your S3 bucket. You can use the AWS Management Console (requires Google Chrome browser) or the AWS CLI to complete this step. If you already have the AWS CLI installed and configured on your local machine, we recommend using that method. Otherwise, use the console if you have the latest version of Google Chrome installed.

<details>
<summary><strong>CLI step-by-step instructions (expand for details)</strong></summary><p>

1. With a file manager, navigate to the folder where the lab content is located and open the **UnicornManager** directory from the **WebApplication/5_OAuth/** folder.

1. Open a terminal window and navigate to the folder where the material for this workshop is located. Navigate to the `WebApplication/5_OAuth/UnicornManager` folder.

1. Open the **js** folder.

1. Using your preferred text editor, open the **config.js** file.

1. From the Cognito User Pools console, copy the client app id for the **UnicornManager** application as the value of the **userPoolClientId** property. You can find the application id in the **アプリクライアント** menu of the Cognito console.

1. Change the value of the **region** property to the region you are using for this workshop. For example, I'm using **eu-west-1**.

1. Still in the Cognito User Pools console, open the **ドメイン名** page and copy the custom prefix in the value for the **authDomainPrefix** property. In our sample, this was `wildrydes-{Your Name}`.

1. Finally, open the CloudFormation console and select the pre-requisites stack we created at the beginning of this lab. With the stack selected, use the bottom section of the window to open the **Outputs** tab. Copy the value of the **WildRydesApiInvokeUrl** output variable to the **invokeUrl** property - this value should look like this: `https://xxxxxxxxx.execute-api.xx-xxxxx-x.amazonaws.com/prod`

1. Next, we need to copy the files we just modified to the S3 bucket that hosts our static website. We created the bucket in step #5 of this lab and it should be called **unicornmanager-&lt;username&gt;**. You can use the AWS CLI or the management console with a compatible browser to upload the files.

##### AWS CLI

1. With a terminal, navigate to the **UnicornManager** directory in the lab material folder.

1. Run the following command:

    ```
    aws s3 sync . s3://YOUR_BUCKET_NAME --region YOUR_BUCKET_REGION
    ```

##### AWS Console

1. Open the **S3** console and select the Unicorn Manager bucket.

1. In the **概要** tab, click the **アップロード** button.

1. From a file browser window, select all of the files in the **UnicornManager** folder and drag them to S3's upload window.

(Original)

1. With a file manager, navigate to the folder where the lab content is located and open the **UnicornManager** directory from the **WebApplication/5_OAuth/** folder.

1. Open a terminal window and navigate to the folder where the material for this workshop is located. Navigate to the `WebApplication/5_OAuth/UnicornManager` folder.

1. Open the **js** folder.

1. Using your preferred text editor, open the **config.js** file.

1. From the Cognito User Pools console, copy the client app id for the **UnicornManager** application as the value of the **userPoolClientId** property. You can find the application id in the **App clients** menu of the Cognito console.

1. Change the value of the **region** property to the region you are using for this workshop. For example, I'm using **us-east-2**.

1. Still in the Cognito User Pools console, open the **Domain name** page and copy the custom prefix in the value for the **authDomainPrefix** property. In our sample, this was `wildrydes-sapessi`.

1. Finally, open the CloudFormation console and select the pre-requisites stack we created at the beginning of this lab. With the stack selected, use the bottom section of the window to open the **Outputs** tab. Copy the value of the **WildRydesApiInvokeUrl** output variable to the **invokeUrl** property - this value should look like this: `https://xxxxxxxxx.execute-api.xx-xxxxx-x.amazonaws.com/prod`

1. Next, we need to copy the files we just modified to the S3 bucket that hosts our static website. We created the bucket in step #5 of this lab and it should be called **unicornmanager-&lt;username&gt;**. You can use the AWS CLI or the management console with a compatible browser to upload the files.
##### AWS CLI

1. With a terminal, navigate to the **UnicornManager** directory in the lab material folder.

1. Run the following command:

    ```
    aws s3 sync . s3://YOUR_BUCKET_NAME --region YOUR_BUCKET_REGION
    ```
##### AWS Console

1. Open the **S3** console and select the Unicorn Manager bucket.

1. In the **Overview** tab, click the **Upload** button.

1. From a file browser window, select all of the files in the **UnicornManager** folder and drag them to S3's upload window.

</p></details>

### Testing the application

新しい Unicorn Manager アプリケーションのWebページを開く前に、ユニコーン用のユーザを作成する必要があります。 **DynamoDB** コンソールを使用して、**テーブル** ページを開き、**Rides** テーブルを選択します。 **項目** タブで、ライドのリストを更新する。 **UnicornName** フィールドから最も一般的なユニコーン名を取り、値をコピーします。

> Before we open the web page for the new Unicorn Manager application, we need to create a user for our unicorn. Using the **DynamoDB** console, open the **Tables** page and select the **Rides** table. In the **Items** tab, refresh the list of rides. Take the most common unicorn name from the **UnicornName** field and copy the value.

次に、手順5で作成した CloudFront ディストリビューションドメインに移動して、ユニコーンマネージャーアプリケーションを開きます。ドメインは **xxxxxxxxxxxx.cloudfront.net** のようになります。 アプリケーションは、ログインしていないことを自動的に検出し、Cognito がホストするログインページにリダイレクトします。 ログインページで、フォームの下部にある **サインアップ** リンクを使用してください。

> Next, open the unicorn manager application by navigating to the CloudFront distribution domain we created in step #5 - the domain should look like this: **xxxxxxxxxxxx.cloudfront.net**. The application detects that we are not logged in an automatically redirects us to the Cognito hosted login page. On the login page, use the **Sign up** link at the bottom of the form.

サインアップページで、DynamoDBテーブルからコピーした **UnicornName** 値をユーザー名、有効な電子メールアドレスとして使用し、ユーザーのパスワードを作成します。 ほとんどの電子メールアドレスでは、**+** の後に接尾辞を付けてカスタムアドレスを作成できます。 たとえば、**youremail+unicorn@emaildomain.com** でサインアップできます。

> In the Sign up page, use the **UnicornName** value we copied from the DynamoDB table as the username, a valid email address, and create a password for the user. With most email addresses you can use a suffix preceded by **+** to create custom addresses. For example, you could sign up with **youremail+unicorn@emaildomain.com**.

![Sign up unicorn screenshot](../images/user-pool-unicorn-signup.png)

ユニコーンアカウントを作成するには、**Sign up** をクリックしてください。 ホストされている登録UIから確認コードの入力を求められます。このコードを電子メールで受け取ったはずです。 確認コードをフォームに貼り付けて、**Confirm account** をクリックしてください。

> Click **Sign up** to create the unicorn account. The hosted registration ui will ask you for the verification code, you should have received this code via email. Paste the verification code in the form and click **Confirm account**.

アカウントが確認されると、アプリケーションは Unicorn manager のメインWebページにあなたをリダイレクトします。 登録したユニコーンのライドのリストをロードするには、右上の **Refresh** ボタンを使用してください。

> Once the account is confirmed, the application will redirect you to the main web page of the Unicorn manager. Use the **Refresh** button on the top right to load a list of the rides for the unicorn you registered.

私たちは **Wild Rydes** をプラットフォームに変えました。 サードパーティの開発者は、新しいクライアントアプリIDを要求し、ホストされたUIを使用して新しいユーザーを認証および登録することができます。 これにより、私たちのチームが独自に生み出すことができるものを超えて、私たちの顧客基盤とツールキットを拡大することができます。

> We have now turned **Wild Rydes** into a platform. Third party developers can now ask us for a new client app id, use our hosted UI to authenticate and register new users. This will allow us to grow our customer base and toolkit beyond what our team can produce by itself, **UnicornManager** is just the first step.

**社内ワークショップメモ: ここまで到達したらこのワークショップは完了です。おつかれさまでした。**
