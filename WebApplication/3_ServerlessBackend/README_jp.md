# Module 3: Serverless Service Backend

このモジュールでは、AWS Lambda と Amazon DynamoDB を使用して、Webアプリケーションからのリクエストを処理するためのバックエンドプロセスを構築します。 最初のモジュールにデプロイしたブラウザアプリケーションを使用すると、ユーザーはユニコーンを自分の好きな場所に手配するようにリクエストできます。 これらの要求を満たすためには、ブラウザで実行されているJavaScriptがクラウドで実行されているサービスを呼び出す必要があります。

> In this module you'll use AWS Lambda and Amazon DynamoDB to build a backend process for handling requests from your web application. The browser application that you deployed in the first module allows users to request that a unicorn be sent to a location of their choice. In order to fulfill those requests, the JavaScript running in the browser will need to invoke a service running in the cloud.

ユーザーがユニコーンをリクエストするたびに呼び出される Lambda 関数を実装します。 この関数は、フリートからユニコーンを選択し、その要求を DynamoDB テーブルに記録してから、手配されているユニコーンに関する詳細情報をフロントエンドアプリケーションに返します。

> You'll implement a Lambda function that will be invoked each time a user requests a unicorn. The function will select a unicorn from the fleet, record the request in a DynamoDB table and then respond to the front-end application with details about the unicorn being dispatched.

![Serverless backend architecture](../images/serverless-backend-architecture.png)

この関数は、Amazon API Gateway を使用してブラウザから呼び出されます。 次のモジュールでその接続を実装します。 このモジュールでは、あなたの関数を単独でテストするだけです。

> The function is invoked from the browser using Amazon API Gateway. You'll implement that connection in the next module. For this module you'll just test your function in isolation.

次のモジュールにスキップしたい場合は、[module 4 (RESTful APIs)](../4_RESTfulAPIs) からスタックを起動できます。

> If you want to skip ahead to the next module, you can **launch the stack from [module 4 (RESTful APIs)](../4_RESTfulAPIs)**.

## Implementation Instructions

以下の各セクションでは、実装の概要と詳細な手順を説明します。 すでにAWSマネジメントコンソールに精通している場合、またはウォークスルーに従わずにサービスを自分で探索したい場合は、概要に実装の完了に十分なコンテキストを記載してください。

> Each of the following sections provide an implementation overview and detailed, step-by-step instructions. The overview should provide enough context for you to complete the implementation if you're already familiar with the AWS Management Console or you want to explore the services yourself without following a walkthrough.

最新バージョンのChrome、Firefox、またはSafari Webブラウザを使用している場合は、セクションを展開するまでステップバイステップの説明は表示されません。

> If you're using the latest version of the Chrome, Firefox, or Safari web browsers the step-by-step instructions won't be visible until you expand the section.

### 1. Create an Amazon DynamoDB Table

Amazon DynamoDB コンソールを使用して新しい DynamoDB テーブルを作成します。 `Rides` テーブルを作り、それに `RideId` というパーティションキーをString型で与えます。 テーブル名とパーティションキーは大文字と小文字を区別します。 提供されたIDを正確に使用してください。 他のすべての設定にはデフォルト値を使用してください。

> Use the Amazon DynamoDB console to create a new DynamoDB table. Call your table `Rides` and give it a partition key called `RideId` with type String. The table name and partition key are case sensitive. Make sure you use the exact IDs provided. Use the defaults for all other settings.

テーブルを作成したら、次の手順で使用する ARN を書き留めます。

> After you've created the table, note the ARN for use in the next step.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. AWSマネジメントコンソールから、サービスを選択し、データベースの下の DynamoDB を選択します。

1. **テーブルの作成** を選択します。

1. **テーブル名** に `Rides-{Your Name}` を入力してください。 このフィールドでは大文字と小文字が区別されます。

1. **パーティションキー** に `RideId` を入力し、キータイプに **文字列** を選択します。 このフィールドでは大文字と小文字が区別されます。

1. **デフォルト設定の使用** ボックスをチェックし、**作成** を選択します。

1. 新しいテーブルの概要セクションの一番下までスクロールして、**ARN** を書き留めます。 次のセクションでこれを使います。

(Original)

1. From the AWS Management Console, choose **Services** then select **DynamoDB** under Databases.

1. Choose **Create table**.

1. Enter `Rides` for the **Table name**. This field is case sensitive.

1. Enter `RideId` for the **Partition key** and select **String** for the key type. This field is case sensitive.

1. Check the **Use default settings** box and choose **Create**.

    ![Create table screenshot](../images/ddb-create-table.png)

1. Scroll to the bottom of the Overview section of your new table and note the **ARN**. You will use this in the next section.

</p></details>


### 2. Create an IAM Role for Your Lambda function

#### Background

すべての Lambda 関数には関連付けられた IAM ロールがあります。 このロールは、その機能が他のどのAWSサービスと連携できるかを定義します。 このワークショップの目的のために、ログを Amazon CloudWatch Logs に書き込む権限と、データを DynamoDB テーブルに書き込む権限を付与する IAM ロールを作成する必要があります。

> Every Lambda function has an IAM role associated with it. This role defines what other AWS services the function is allowed to interact with. For the purposes of this workshop, you'll need to create an IAM role that grants your Lambda function permission to write logs to Amazon CloudWatch Logs and access to write items to your DynamoDB table.

#### High-Level Instructions

IAM コンソールを使用して新しい役割を作成します。 `WildRydesLambda` という名前を付け、ロールタイプとして AWS Lambda を選択します。 Amazon CloudWatch Logs に書き込み、DynamoDB テーブルにデータを追加するための機能権限を付与するポリシーをアタッチする必要があります。

> Use the IAM console to create a new role. Name it `WildRydesLambda` and select AWS Lambda for the role type. You'll need to attach policies that grant your function permissions to write to Amazon CloudWatch Logs and put items to your DynamoDB table.


必要な CloudWatch Logs アクセス許可を付与するには、`AWSLambdaBasicExecutionRole` という管理ポリシーをこのロールにアタッチします。 また、前のセクションで作成したテーブルに対して `ddb:PutItem` アクションを許可する、このロール用のカスタムインラインポリシーを作成します。

> Attach the managed policy called `AWSLambdaBasicExecutionRole` to this role to grant the necessary CloudWatch Logs permissions. Also, create a custom inline policy for your role that allows the `ddb:PutItem` action for the table you created in the previous section.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. AWSマネジメントコンソールから、**サービス** をクリックし、「セキュリティ、アイデンティティ、コンプライアンス」セクションで **IAM** を選択します。

1. 左側のナビゲーションバーで **ロール** を選択し、**ロールの作成** を選択します。

1. **AWS サービス** グループからロールタイプとして **Lambda** を選択し、次に **次のステップ: アクセス権限** をクリックします。

    **Note:** ロールタイプを選択すると、AWSサービスがあなたに代わってこのロールを引き受けることを許可するあなたのロール用の信頼ポリシーが自動的に作成されます。 CLI、AWS CloudFormation、またはその他のメカニズムを使用してこのロールを作成している場合は、信頼ポリシーを直接指定します。

1. **ポリシーのフィルタ** テキストボックスに `AWSLambdaBasicExecutionRole` と入力し、そのロールの横にあるチェックボックスをオンにします。

1. **次のステップ: タグ**、**次のステップ: 確認** をクリックします。

1. **ロール名** に `WildRydesLambda-{Your Name}` と入力してください。

1. **ロールの作成** を選択します。

1. ロールページのフィルタボックスに `WildRydesLambda-{Your Name}` と入力し、作成したロールを選択してください。

1. アクセス権限タブで、新しいインラインポリシーを作成するために右隅にある **インラインポリシーの追加** リンクを選択します。

1. **サービスの選択** を選択してください。

1. **サービスの検索** というラベルの付いた検索ボックスに `DynamoDB` と入力し、表示されたら **DynamoDB** を選択します。

1. **アクションの選択** を選択してください。

1. **フィルタアクション** というラベルの付いた検索ボックスに `PutItem` と入力し、表示されたら **PutItem** の横のチェックボックスをオンにします。

1. **リソース** セクションを選択してください。

1. **指定** オプションを選択した状態で、**table** セクションの **ARN の追加** リンクを選択します。

1. 前のセクションで作成したテーブルのARNを **DynamoDB_table の ARN の指定** フィールドに貼り付け、**追加** を選択します。

1. **ポリシーの確認** を選択してください。

1. ポリシーの名前に `DynamoDBWriteAccess-{Your Name}` と入力し、**ポリシーの作成** を選択します。

(Original)

1. From the AWS Management Console, click on **Services** and then select **IAM** in the Security, Identity & Compliance section.

1. Select **Roles** in the left navigation bar and then choose **Create new role**.

1. Select **Lambda** for the role type from the **AWS service** group, then click **Next: Permissions**

    **Note:** Selecting a role type automatically creates a trust policy for your role that allows AWS services to assume this role on your behalf. If you were creating this role using the CLI, AWS CloudFormation or another mechanism, you would specify a trust policy directly.

1. Begin typing `AWSLambdaBasicExecutionRole` in the **Filter** text box and check the box next to that role.

1. Click **Next: Review**.

1. Enter `WildRydesLambda` for the **Role name**.

1. Choose **Create role**.

1. Type `WildRydesLambda` into the filter box on the Roles page and choose the role you just created.

1. On the Permissions tab, choose the **Add inline policy** link in the lower right corner to create a new inline policy.
    ![Inline policies screenshot](../images/inline-policies.png)

1. Select **Choose a service**.

1. Begin typing `DynamoDB` into the search box labeled **Find a service** and select **DynamoDB** when it appears.
    ![Select policy service](../images/select-policy-service.png)

1. Choose **Select actions**.

1. Begin typing `PutItem` into the search box labeled **Filter actions** and check the box next to **PutItem** when it appears.

1. Select the **Resources** section.

1. With the **Specific** option selected, choose the Add ARN link in the **table** section.

1. Paste the ARN of the table you created in the previous section in the **Specify ARN for table** field, and choose **Add**.

1. Choose **Review Policy**.

1. Enter `DynamoDBWriteAccess` for the policy name and choose **Create policy**.
    ![Review Policy](../images/review-policy.png)

</p></details>

### 3. Create a Lambda Function for Handling Requests

#### Background

AWS Lambda は、HTTPリクエストなどのイベントに応答してコードを実行します。 このステップでは、WebアプリケーションからのAPI要求を処理してユニコーンを手配するコア機能を構築します。 次のモジュールでは、Amazon API Gateway を使用して、ユーザーのブラウザから呼び出すことができるHTTPエンドポイントを公開するRESTful APIを作成します。 次に、このステップで作成した Lambda 関数をそのAPIに接続して、Webアプリケーション用のバックエンドを作成します。

> AWS Lambda will run your code in response to events such as an HTTP request. In this step you'll build the core function that will process API requests from the web application to dispatch a unicorn. In the next module you'll use Amazon API Gateway to create a RESTful API that will expose an HTTP endpoint that can be invoked from your users' browsers. You'll then connect the Lambda function you create in this step to that API in order to create a fully functional backend for your web application.

#### High-Level Instructions

AWS Lambda コンソールを使用して、APIリクエストを処理する `RequestUnicorn` という新しい Lambda 関数を作成します。 提供されている [requestUnicorn.js](requestUnicorn.js) サンプル実装を使用してください。 そのファイルから AWS Lambda コンソールのエディターにコピーして貼り付けるだけです。

> Use the AWS Lambda console to create a new Lambda function called `RequestUnicorn` that will process the API requests. Use the provided [requestUnicorn.js](requestUnicorn.js) example implementation for your function code. Just copy and paste from that file into the AWS Lambda console's editor.

前のセクションで作成した `WildRydesLambda` IAMロールを使用するように設定してください。

> Make sure to configure your function to use the `WildRydesLambda` IAM role you created in the previous section.

<details>
<summary><strong>Step-by-step instructions (expand for details)</strong></summary><p>

1. **サービス** を選択し、コンピューティングセクションで **Lambda** を選択します。

1. **関数の作成** をクリックします。

1. デフォルトの **一から作成** カードを選択したままにします。

1. **関数名** フィールドに `RequestUnicorn-{Your Name}` と入力します。

1. **ランタイム** として **Node.js 8.10** を選択します。

1. **実行ロール** ドロップダウンから `既存のロールを使用する` が選択されていることを確認します。

1. **既存のロール** ドロップダウンから `WildRydesLambda-{Your Name}` を選択します。

1. **関数の作成** をクリックします。

1. **関数コード** セクションまでスクロールし、コードエディタ内にある **index.js** を [requestUnicorn.js](requestUnicorn.js) の内容に置き換えます。

1. 上記作業後の **index.js** の90行目の `TableName: 'Rides',` の `Rides` を `Rides-{Your Name}` のように前のセクションで作成した DynamoDB テーブル名と一致するように編集します。

1. ページの右上隅にある **保存** をクリックします。

(Original)

1. Choose on **Services** then select **Lambda** in the Compute section.

1. Click **Create function**.

1. Keep the default **Author from scratch** card selected.

1. Enter `RequestUnicorn` in the **Name** field.

1. Select **Node.js 6.10** for the **Runtime**.

1. Ensure `Choose an existing role` is selected from the **Role** dropdown.

1. Select `WildRydesLambda` from the **Existing Role** dropdown.
    ![Create Lambda function screenshot](../images/create-lambda-function.png)

1. Click on **Create function**.

1. Scroll down to the **Function code** section and replace the existing code in the **index.js** code editor with the contents of [requestUnicorn.js](requestUnicorn.js).
    ![Create Lambda function screenshot](../images/create-lambda-function-code.png)

1. Click **"Save"** in the upper right corner of the page.

</p></details>

## Implementation Validation

このモジュールでは、AWS Lambda コンソールを使用して構築した関数をテストします。 次のモジュールでは、最初のモジュールでデプロイしたブラウザベースのアプリケーションから関数を呼び出すことができるように、API Gateway を使用してREST APIを追加します。

> For this module you will test the function that you built using the AWS Lambda console. In the next module you will add a REST API with API Gateway so you can invoke your function from the browser-based application that you deployed in the first module.

1. 関数の編集画面で、**テストイベントの選択** から **テストイベントの設定** を選択します。

1. **新しいテストイベントの作成** を選択したままにします。

1. **イベント名** フィールドに `TestRequestEvent-{Your Name}` と入力します。

1. (**社内ワークショップメモ: もしハイフンの入力が禁止されていたら、よしなに命名してください。**)

1. 次のテストイベントをコピーしてエディタに貼り付けます。 `{Your Name}` のところをスペース無しのあなたの名前に置き換えることを忘れないでください。

    ```JSON
    {
        "path": "/ride",
        "httpMethod": "POST",
        "headers": {
            "Accept": "*/*",
            "Authorization": "eyJraWQiOiJLTzRVMWZs",
            "content-type": "application/json; charset=UTF-8"
        },
        "queryStringParameters": null,
        "pathParameters": null,
        "requestContext": {
            "authorizer": {
                "claims": {
                    "cognito:username": "the_username"
                }
            }
        },
        "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
    }
    ```

1. **作成** をクリックします。

1. メインの関数エディタで、ドロップダウンから `TestRequestEvent-{Your Name}` を選択した状態で **テスト** をクリックします。

1. ページの一番上までスクロールして、**実行結果** セクションの **詳細** セクションを展開します。

1. 実行が成功したこと、および関数の結果が次のようになっていることを確認してください。

    ```JSON
    {
        "statusCode": 201,
        "body": "{\"RideId\":\"SvLnijIAtg6inAFUBRT+Fg==\",\"Unicorn\":{\"Name\":\"Rocinante\",\"Color\":\"Yellow\",\"Gender\":\"Female\"},\"Eta\":\"30 seconds\"}",
        "headers": {
            "Access-Control-Allow-Origin": "*"
        }
    }
    ```

(Original)

1. From the main edit screen for your function, select **Configure test event** from the the **Select a test event...** dropdown.
    ![Configure test event](../images/configure-test-event.png)

1. Keep **Create new test event** selected.

1. Enter `TestRequestEvent` in the **Event name** field

1. Copy and paste the following test event into the editor:

    ```JSON
    {
        "path": "/ride",
        "httpMethod": "POST",
        "headers": {
            "Accept": "*/*",
            "Authorization": "eyJraWQiOiJLTzRVMWZs",
            "content-type": "application/json; charset=UTF-8"
        },
        "queryStringParameters": null,
        "pathParameters": null,
        "requestContext": {
            "authorizer": {
                "claims": {
                    "cognito:username": "the_username"
                }
            }
        },
        "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
    }
    ```

    ![Configure test event](../images/configure-test-event-2.png)

1. Click **Create**.

1. On the main function edit screen click **Test** with `TestRequestEvent` selected in the dropdown.   

1. Scroll to the top of the page and expand the **Details** section of the **Execution result** section.

1. Verify that the execution succeeded and that the function result looks like the following:

    ```JSON
    {
        "statusCode": 201,
        "body": "{\"RideId\":\"SvLnijIAtg6inAFUBRT+Fg==\",\"Unicorn\":{\"Name\":\"Rocinante\",\"Color\":\"Yellow\",\"Gender\":\"Female\"},\"Eta\":\"30 seconds\"}",
        "headers": {
            "Access-Control-Allow-Origin": "*"
        }
    }
    ```

Lambda コンソールを使用して新しい関数を正常にテストできたら、次のモジュール [RESTful APIs](../4_RESTfulAPIs) に進むことができます。

> After you have successfully tested your new function using the Lambda console, you can move on to the next module, [RESTful APIs](../4_RESTfulAPIs).
