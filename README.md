
## Overview

Your Discord token is sensitive information that must **NOT** be placed in your repo.  In 244, we placed our token in a `.env` file and used the [dotenv-java library](https://github.com/cdimascio/dotenv-java) to load the value in code.  When we automate the creation and deployment of our Discord bot, we cannot automate the creation of a `.env` file because that will expose the token publicly.

Instead of using a `.env` file, we will store our Discord token in [SecretsManager](https://aws.amazon.com/secrets-manager/) and use the [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/) to retrieve the token.



## Create a Secret for the Discord Token

[Secrets Manager](https://aws.amazon.com/secrets-manager/) is the AWS service that allows users to store sensitive information.  Each secret has a *name* and is stored as a *key/value* pair.  

We will create a *secret* named `220_Discord_Token` with a *key* named `DISCORD_TOKEN` and a *value* of the token.


* Open the AWS Console, and go to Secrets Manager
* Click "Store a new Secret"
* Select "Other type of secret"
  * Key: `DISCORD_TOKEN`
  * Value: `<your token>`
* Click "Next" to go to the Configure secret page
  * Secret Name: `220_Discord_Token`
* Click "Next" through all remaining screens - the default values are appropriate.
* Click "Store" to save the secret


You can ignore the error message shown.  If you reload the page, you will see your secret.


## View the Secret on AWS


After you have saved a secret, you can retrieve it at any time (unlike Github Action secrets).

* Click on your secret name to load a page about the secret
* In the "Secret Value" section, click "Retrieve secret value" to see the key/value pair.



## Local Setup

The AWS SDK for Java uses credentials to authenticate, and the AWS Learner Lab creates *temporary credentials* each time you launch the lab.  We will store our credentials on our laptop in the file `~/.aws/credentials`.

**You will need to change your credentials every time you work on your project**.  This is a side effect of the way the Learner Lab is implemented.  In a full AWS account, you can create permanent credentials.

* Create a folder named `~/.aws` (one-time operation)

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

We also need to configure Maven to use a different version of the compiler plugin.  The default uses an older compiler that is not compatible with modern Java language features.

* In the `<plugins>` section of the `pom.xml` add

```
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
      </plugin>
```

Finally, we will need to specify the version of Java to use.  We need to ensure that this version of Java is available both in development (on your laptop) and in production (an an EC2 instance).

We will use version 21 of Java.  In the `<properties>` section (near the top of the `pom.xml` file) add the following line or change the version if it is present:

```
    <maven.compiler.release>21</maven.compiler.release>
```


## Java Code


In `cs220s25`, the repo [`aws_secrets_java`](https://github.com/cs220s25/aws_secrets_java) contains a demo program on how to load the secret from AWS Secrets Manager.

* The `Secrets` class contains the code to interact with AWS Secrets Manager and to parse the results
* If an exception is thrown, it is caught and a `SecretsException` is thrown.
* The `Main` class contains code to show usage of the class.



## Local Deploy


Before you try to deploy this code on an EC2 instance, make sure you can run it on your laptop.

```
mvn package
java -jar target/secretsDemo-1.0.0-jar-with-dependencies.jar
```

If everything is correct, the program will output the secret you saved in Secrets Manager.



## EC2 Deploy


To run this system on an EC2 instance we need to:

* Ensure that the EC2 instance has permission to access Secret Manager
* Install Java and Maven
* Clone the repo
* Build the application


#### Launch the Instance with Permissions to Access Secrets Manager


AWS has a service called [Identity and Access Management (IAM)](https://docs.aws.amazon.com/iam/) that contains the concept of a [*role*](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html).  We will *attach* the `LabRole` to our EC2 instance to give it permissions to access Secrets Manager.  Once this role is attached, we will not need to save credentials in an `.aws/credentials` file on the EC2 instance.

* To attach the LabRole, when you launch the EC2 instance, go to "Advanced details" section under "IAM instance profile" and select "LabInstanceProfile".


![instance profile](https://i.ibb.co/Y7KH8qbD/f32cc23777f7.png)


#### Install Java and Maven

The Java language has multiple versions, and Maven can support different versions of Java.  When we deploy an application we must make sure that the `pom.xml` file, the version of Java, and the version of Maven are all compatible.

We will install a version of Maven that comes bundled with [Corretto 21](https://docs.aws.amazon.com/corretto/latest/corretto-21-ug/what-is-corretto-21.html), an AWS distribution of the OpenJDK.  This will ensure that only **ONE** version of Java is installed on the EC2 instance, which reduces possible version conflicts.

```
sudo yum install -y maven-amazon-corretto21
```

As discussed earlier, the `pom.xml` file specifies that we are using Java version 21, the same version installed by Maven.


#### Clone the Repo

This is similar to what we have done in previous EC2 deployments.


```
sudo yum install -y git
git clone https://github.com/cs220s25/aws_secrets_java.git
```

#### Build and Run the Application


Now that we have Java, Maven, and the source repo, we can build and run the application on the EC2 instance.

NOTE:  We do **NOT** need to create a `.aws/credentials` file because the `LabRole` attached to the instance will grant permissions to Secrets Manager.


```
mvn package
java -jar target/secretsDemo-1.0.0-jar-with-dependencies.jar
```

The output of this program will be the value you stored in Secrets Manager.




## References

* [AWS Docs: Set up an Apache Maven project](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/setup-project-maven.html)
* [AWS Docs: Get a Secrets Manager secret value using the Java AWS SDK](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets-java-sdk.html)
* [AWS Docs: Amazon Corretto](https://aws.amazon.com/corretto/?filtered-posts.sort-by=item.additionalFields.createdDate&filtered-posts.sort-order=desc)
