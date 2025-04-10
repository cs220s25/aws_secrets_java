
## Overview

Your Discord token is sensitive information that must **NOT** be placed in your repo.  In 244, we placed our token in a `.env` file and used the [dotenv-java library](https://github.com/cdimascio/dotenv-java) to load the value in code.  When we automate the creation and deployment of our Discord bot, we cannot automate the creation of a `.env` file because that will expose the token publicly.

Instead of using a `.env` file, we will store our Discord token in [SecretsManager](https://aws.amazon.com/secrets-manager/) and use the [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/) to retrieve the token.


## Setup

The AWS SDK for Java uses credentials to authenticate.  The AWS Learner Lab creates *temporary credentials* each time you launch the lab.

**You will need to change your credentials every time you work on your project**.  This is a side effect of the way the Learner Lab is implemented.  In a full AWS account, you can create permanent credentials.

* Create a folder named `~/.aws`

  ```
  mkdir ~/.aws
  ```
  
* Start the the Learner Lab and then click on "AWS Details" at the top of the lab interface.  Copy the text in the window below "AWS CLI"

```
[default]
aws_access_key_id=ASIAVBIXLOLWZGSECRET
aws_secret_access_key=gyud18iuUSACCESSKEY
aws_session_token=IQoJb3JpZ2luX2VjECsaCXVzLXdlc3QtMiJHMEUCIFNJRpsBQBxwT+nRg1vX7xAFN7zSmvU/OvW9kbS9M1lFAiEAt3PQREALLY_LONG_TOKEN
```

* Paste this string into the file `~/.aws/credentials`.


## Create a Secret for the Discord Token


* Open the AWS Console, and go to Secrets Manager
* Click "Store a new Secret"
* Select "Other type of secret"
  * Key: `DISCORD_TOKEN`
  * Value: <your token>
* Click "Next" to go to the Configure secret page
  * Secret Name: 220_Discord_Token
* Click "Next" through all remaining screens - the default values are appropriate.
* Click "Store" to save the secret


You can ignore the error message shown.  If you reload the page, you will see your secret.


## View the Secret on AWS

* Click on your secret name to load a page about the secret
* In the "Secret Value" section, click "Retrieve secret value" to see the key/value pair.


## POM Configuration

We need to include 2 dependencies in our POM, one for the AWS SDK and the other for a library to parse the key.

* AWS SDK for Java - SecretsManager

```
    <dependency>
      <groupId>software.amazon.awssdk</groupId>
      <artifactId>secretsmanager</artifactId>
      <version>2.31.17</version>
    </dependency>
```

* The Jackson Databind library to parse the key/value pair

```
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.18.2</version>
    </dependency>
```


## Java Code

In `cs220s25`, the repo `aws_secrets_java` contains a demo program on how to load the secret from AWS Secrets Manager

* The `Secrets` class contains the code to interact with AWS Secrets Manager and to parse the results
* If an exception is thrown, it is caught and a `SecretsException` is thrown.
* The `Main` class contains code to show usage of the class.



## References

* [AWS Docs: Set up an Apache Maven project](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/setup-project-maven.html)
* [AWS Docs: Get a Secrets Manager secret value using the Java AWS SDK](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets-java-sdk.html)