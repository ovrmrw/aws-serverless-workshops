# Wild Rydes Serverless Workshops

このリポジトリはワークショップの集まりとその他の構築のガイドに役立つコンテンツです。
様々なサーバーレスアプリケーションは AWS Lambda, Amazon API Gateway, Amazon DynamoDB, AWS Step Functions, Amazon Kinesis そしてその他のサービスを使用します。

> This repository contains a collection of workshops and other hands on content that will guide you through building ?   various serverless applications using AWS Lambda, Amazon API Gateway, Amazon DynamoDB, AWS Step Functions, Amazon Kinesis, and other services.

# Workshops

- [**ウェブアプリケーション**](WebApplication) - このワークショップは動的なサーバーレスアプリケーションの構築する方法を説明します。S3 で静的リソースをホスティングする方法、Amazon Cognito でユーザー管理と認証を行う方法、Amazon API Gateway と AWS Lambda と Amazon DynamoDB を用いてバックエンドの RESTful API を構築する方法を学びます。

- [**認証**](Auth) - このワークショップはアプリケーションのサインアップとサインイン機能から始めて、複数層のセキュリティで構築する方法、サーバーレスのマイクロサービスをセキュアにする方法、アプリケーションのユーザーにきめ細かなアクセスコントロールを提供する IAM を使用する方法を紹介します。AWS Amplify を Amazon Cognito, Amazon API Gateway, AWS Lambda および IAM に連携して、組み込まれた認証・認可を提供する方法を学びます。

- [**データ処理**](https://dataprocessing.wildrydes.com) - このワークショップはサーバーレスアプリケーションを用いたデータの収集・蓄積・処理をする方法のデモンストレーションです。このワークショップ内では Amazon Kinesis Data Streams and Amazon Kinesis Data Analytics を使ったリアルタイムストリーミングアプリケーションを構築する方法や、Amazon Kinesis Data Firehose と S3 を用いたデータストリームをアーカイブする方法、そしてそれらのファイルに対して Amazon Athena を用いたアドホックなクエリの実行方法を学びます。

- [**DevOps**](DevOps) - このワークショップでは [Serverless Application Model (SAM)](https://github.com/awslabs/serverless-application-model) を使った Amazon API Gateway と AWS Lambda と Amazon DynamoDB でのサーバーレスアプリケーションを構築する方法を紹介します。ワークステーションからアプリケーションの更新リリースまでの SAM の利用方法、AWS CodePipeline と AWS CodeBuild を用いた サーバーレスアプリケーションの CI/CD パイプラインの構築方法、アプリケーションの複数環境を管理するためのパイプラインを強化する方法を学びます。

- [**画像処理**](ImageProcessing) - ここではバックエンドでの協調したワークフローを使ってサーバーレスで画像処理するアプリケーションを構築する方法を紹介します。Amazon Rekogntion のディープラーニングに基づく顔認証機能を活用しつつ、複数の AWS Lambda 関数 が協調する AWS Step Functions の基本を学びます。

- [**Multi Region**](MultiRegion) - このワークショップでは2つのリージョンにレプリケーションして障害発生時に自動でフェイルオーバーするサーバーレスのチケットシステムを構築する方法を紹介します。AWS Lambda 関数のデプロイのいろはを、API Gateway での公開および Route53 と DynamoDB のレプリケーションの設定を通して学びます。

- [**セキュリティ**](https://github.com/aws-samples/aws-serverless-security-workshop) - このワークショップでは AWS Lambda・Amazon API Gateway・RDS Aurora でのサーバーレスアプリケーション構築をセキュアに行うテクニックを紹介します。アイデンティティとアクセス管理・インフラストラクチャー・データ・コード・ロギングと監視 の5つの領域について、サーバーレスアプリケーションのセキュリティを向上に活用できるAWSサービスと機能について紹介します。

(Original)

- [**Web Application**](WebApplication) - This workshop shows you how to build a dynamic, serverless web application. You'll learn how to host static web resources with Amazon S3, how to use Amazon Cognito to manage users and authentication, and how to build a RESTful API for backend processing using Amazon API Gateway, AWS Lambda and Amazon DynamoDB.

- [**Auth**](Auth) - This workshop shows you how to build in security at multiple layers of your application, starting with sign-up and sign-in functionality for your application, how to secure serverless microservices, and how to leverage AWS's identity and access management (IAM) to provide fine-grained access control to your application's users. You'll learn how AWS Amplify integrates with Amazon Cognito, Amazon API Gateway, AWS Lambda, and IAM to provide an integrated authentication and authorization experience.

- [**Data Processing**](https://dataprocessing.wildrydes.com) - This workshop demonstrates how to collect, store, and process data with a serverless application. In this workshop you'll learn how to build real-time streaming applications using Amazon Kinesis Data Streams and Amazon Kinesis Data Analytics, how to archive data streams using Amazon Kinesis Data Firehose and Amazon S3, and how to run ad-hoc queries on those files using Amazon Athena.

- [**DevOps**](DevOps) - This workshop shows you how to use the [Serverless Application Model (SAM)](https://github.com/awslabs/serverless-application-model) to build a serverless application using Amazon API Gateway, AWS Lambda, and Amazon DynamoDB. You'll learn how to use SAM from your workstation to release updates to your application, how to build a CI/CD pipeline for your serverless application using AWS CodePipeline and AWS CodeBuild, and how to enhance your pipeline to manage multiple environments for your application.

- [**Image Processing**](ImageProcessing) - This module shows you how to build a serverless image processing application using workflow orchestration in the backend. You'll learn the basics of using AWS Step Functions to orchestrate multiple AWS Lambda functions while leveraging the deep learning-based facial recognition features of Amazon Rekogntion.

- [**Multi Region**](MultiRegion) - This workshop shows you how to build a serverless ticketing system that is replicated across two regions and provides automatic failover in the event of a disaster. You will learn the basics of deploying AWS Lambda functions, exposing them via API Gateway, and configuring replication using Route53 and DynamoDB streams.

- [**Security**](https://github.com/aws-samples/aws-serverless-security-workshop) - This workshop shows you techniques to secure a serverless application built with AWS Lambda, Amazon API Gateway and RDS Aurora. We will cover AWS services and features you can leverage to improve the security of a serverless applications in 5 domains: identity & access management, infrastructure, data, code, and logging & monitoring.