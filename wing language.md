DEMO LINK:

https://drive.google.com/file/d/1OKie5DjQl1b6Z3ul8njVCcrw-eVoUx9j/view?usp=sharing

### Simplifying Cloud Deployment with Wing: A Unified Approach to Infrastructure and Application Code

Deploying applications to the cloud often presents numerous challenges for developers. These challenges stem from the need to manage applications while simultaneously writing infrastructure-level code. However, there are various tools and methods available to help developers manage these complexities. For instance, tools like Terraform use configuration languages such as HCL (HashiCorp Configuration Language) to automate infrastructure provisioning.

What developers need is a tool or language that integrates both infrastructure and application code, enabling them to focus more on building and less on managing. This blog introduces you to [Winglang](https://www.winglang.io/), a cloud-oriented programming language that combines infrastructure and runtime code into a single, cohesive unit.

## Common Cloud Deployment Challenges 

Developers encounter several challenges when deploying cloud-based applications. Cloud applications require managing both application code and the underlying infrastructure, which introduces unique complexities. Whether dealing with simple or complex, fully containerized setups, developers must navigate issues related to infrastructure and application logic. Below are some of the key challenges:

1. **Complex Multi-Cloud Management**
   Managing applications across multiple cloud platforms like AWS, Azure, and Google Cloud is challenging due to differing APIs, services, and databases.

2. **Bridging Infrastructure with Application Code**
   Traditionally, developers managed infrastructure using separate tools and languages, increasing the risk of errors and complicating the development process.

3. **Scalability Concerns**
   Scaling applications to meet growing demand, and scaling them down when demand decreases, has traditionally required manual configuration management, which is time-consuming and prone to errors.

4. **Environmental Inconsistencies**
   Ensuring consistent performance across development, testing, and production environments is a frequent challenge. Inconsistencies between these environments can lead to unexpected behavior and bugs when deploying to production.

5. **Cost Management**
   Managing the costs associated with multiple cloud services can be difficult without deep technical knowledge, often resulting in inefficiencies and overspending.

## Winglang: Unifying Infrastructure and Application Code

**Wing** is a cloud-oriented programming language designed to help developers build distributed systems that fully leverage the power of the cloud without the complexity of managing infrastructure. Below is an example of simple Wing code that demonstrates how developers can write code in a straightforward manner:

- A **queue** collects incoming messages.
- An **atomic counter** ensures each message receives a unique identifier.
- A **bucket** stores each message as an object with a unique key derived from the counter.

```javascript
bring cloud;

let queue = new cloud.Queue(timeout: 2m);
let bucket = new cloud.Bucket();
let counter = new cloud.Counter(initial: 100);

queue.setConsumer(inflight (body: str) => {
  let next = counter.inc();
  let key = "myfile-{next}.txt";
  bucket.put(key, body);
});
```

## Winglang: A Language for the Cloud

Wing addresses several pain points for developers and organizations:

1. **Reduced Deployment Time**: Wing significantly speeds up deployment and simplifies the testing and debugging of cloud code.
2. **Local Development Environment**: Wing provides a local development environment with simulation, visualization, and fast reloading capabilities, allowing developers to test and debug applications without deploying them to the cloud.
3. **Multi-Cloud Support**: Wing supports multi-cloud deployments, enabling organizations to use the best features and pricing of different cloud providers, thereby reducing operational risk.

## Building Cloud-Ready Apps with Winglang

Winglang simplifies the process of building cloud-ready applications by integrating infrastructure and runtime code. In this section, we'll walk through creating a Slack App from scratch using Wing.

### Step 1: Setting Up Winglang Locally

Setting up Winglang is straightforward. Follow the [installation guide](https://www.winglang.io/docs/) to complete the setup.

### Step 2: Creating an Empty Project

Set up a new Wing environment with the following command:
```
wing new empty 
Installing dependencies...

Created a new empty project in the current directory! 🎉

Not sure where to get started? In your Wing application folder, try running:

  wing compile - build your project
  wing it - simulate your app in the Wing Console
  wing test - run all tests

Visit the docs for examples and tutorials: https://winglang.io/docs
```
This command installs all dependencies and creates a new empty project.

### Step 3: Exploring the Winglang Slack Library

Creating a Slack app is easy with Winglang. Install the Winglang Slack library using:
```
npm i @winglibs/slack
```
The following code snippet creates an event listener on the **slackbot** object to handle **app_mentions** events:

```javascript
bring slack;
bring cloud;

let myBotToken = new cloud.Secret(name: "mybot_token");
let slackbot = new slack.App(token: myBotToken);

slackbot.onEvent("app_mention", inflight (ctx, event) => {
  let message = new slack.Message();
  message.addSection({
    fields: [
      {
        type: slack.FieldType.mrkdwn,
        text: "*Wow this is markdown!"
      }
    ]
  });

  ctx.channel.postMessage(message);
});
```

When the bot is mentioned in a Slack channel, it calls for few operations:

- **Handler Function (inflight)**: takes two parameters:
    - **ctx (context)**: Provides methods and properties related to the event context, such as interacting with the Slack channel where the event occurred.
    - **event**: Contains the data of the event from Slack.
- **Inside the event handler**:
    - A new **slack.Message** object is created.
    - A section is added to the message using **message.addSection** with a **type** markdown field specified by **slack.FieldType.mrkdwn** and **text**.
    - Finally, the message is posted to the channel from which the event was triggered, using **ctx.channel.postMessage(message)**.

You can explore more about different libraries by visiting [winglang/winglibs](https://github.com/winglang/winglibs) GitHub Repository.

### Step 4: Creating a Slack App

We’ll modify the previous code so that the Slack bot sends a message to a specified channel whenever a file is uploaded to a cloud bucket.

Add the following operations to the existing code:

- Create a cloud storage bucket that serves as an inbox for processing incoming files.
  
  ```javascript
  let inbox = new cloud.Bucket() as "file-inbox";
  ```

- Add an event handler with an **inflight** function to handle new file uploads asynchronously.
  
  ```javascript
  inbox.onCreate(inflight (file_name) => {
    let channel = slackBot.channel("#Slack-Channel-Name");
    channel.post("{file_name} file has been uploaded to inbox.");
  });
  ```

Change the **Slack-Channel-Name** to the name of the channel you want the 
    bot to deliver updates.    
    
Before testing the code, setup [Slack API Dashboard](https://api.slack.com/apps) following the [Slack API Guide](https://api.slack.com/quickstart).

Select OAuth and permissions from sidebar and make sure to provide the below permission to your slack app:

- **app_mentions:read**
- **chat:write**
- **chat:write.public**

![image](https://hackmd.io/_uploads/S1v-97XjA.png)


Install the tokens in your workspace and allow access
![image](https://hackmd.io/_uploads/SJZJq7Xo0.png)

Copy the token generated
![Screenshot 2024-08-21 at 2.20.30 PM](https://hackmd.io/_uploads/rkPh9mQoA.jpg)


### Step 5: Configure Secrets and Test Locally

Add the bot token to your application by running:
```
wing secrets

1 secret(s) found

? Enter the secret value for mybot_token: [hidden]
```
Then, paste the token when prompted. Run the app locally using:
```
wing it
```
Visit http://localhost:3000/ to see a topology view of your application.
![image](https://hackmd.io/_uploads/rk0laQXsC.png)

Test the working by invoking some message and once you click the invoke button, you will a message such as below in your slack channel
![image](https://hackmd.io/_uploads/Bk4f6XmsC.png)

You can also upload file from your local system
![image](https://hackmd.io/_uploads/HyIyiX7oC.png)

To view the list of file in the inbox, enable events subscriptions from your Slack API Dashboard and add **app_mention** under Subscribe to bot events section.

![image](https://hackmd.io/_uploads/BkotTQQsC.png)

Head over to wing console and expose the endpoints of **file-inbox** bucket. After exposing the endpoints copy the link
![image](https://hackmd.io/_uploads/ryn0aXXjA.png)

Append the link with **slack/events** and paste it under the **Request URL** section
![image](https://hackmd.io/_uploads/rkTfR7mjC.png)

Now mention the bot in your channel with **list inbox** message and it will display all the files in the inbox
![image](https://hackmd.io/_uploads/Hy03CQXjC.png)


### Complete Configuration of the `main.w` File

```javascript
bring cloud;
bring slack;

let myBotToken = new cloud.Secret(name: "mybot_token");
let slackBot = new slack.App(token: myBotToken);
let inbox = new cloud.Bucket() as "file-inbox";

inbox.onCreate(inflight (file_name) => {
  let channel = slackBot.channel("#Slack-Channel-Name");
  channel.post("{file_name} file has been uploaded to inbox.");
});

slackBot.onEvent("app_mention", inflight(ctx, event) => {
  let eventText = event["event"]["text"].asStr();
  if eventText.contains("list inbox") {
    let files = inbox.list();
    let message = new slack.Message();
    message.addSection({
      fields: [
        {
          type: slack.FieldType.mrkdwn,
          text: "*Current Inbox:*\n-{files.join("\n-")}"
        }
      ]
    });
    ctx.channel.postMessage(message);
  }
});
```

## Key Features of Wing

1. **Iteration Speed**: Wing's local cloud simulator allows for rapid testing and quick iterations, enabling developers to see the effects of changes almost instantly without requiring cloud deployment.
2. **High-Level Cloud Primitives**: Wing’s Cloud Library provides rich, high-level, cloud-portable resources, enabling developers to create fully functional cloud applications with minimal infrastructure knowledge.
3. **Infrastructure as Policy**: Wing manages infrastructure concerns like networking and security as policies applied across the system, rather than embedding them within application code.
4. **Local Cloud Simulator**: Wing supports running applications on a single host with a local simulator, allowing developers to develop and test cloud applications functionally without deploying them to the cloud.

## Wing vs Terraform: A Comparison

| **Feature**                      | **Wing**                                        | **Terraform**                                   |
|----------------------------------|-------------------------------------------------|-------------------------------------------------|
| **Scope**                        | Integrates infrastructure and application code. | Focuses mainly on infrastructure management.    |
| **Cloud Portability**            | Code can be compiled for any cloud.              | Requires specific configuration files for each cloud.|
| **Development Environment**      | Local simulator for testing.                    | Requires cloud deployment for testing.          |
| **Automatic Configurations**     | Automatically generates infrastructure definitions like IAM policies. | Manual definition of infrastructure components.  |
| **Multi-Cloud and Extensibility**| Single codebase for multiple clouds, supports custom resources. | Separate configurations needed for each cloud, uses custom modules. |

## Getting Started

![Screenshot 2024-08-21 at 2.30.20 PM](https://hackmd.io/_uploads/Bkgnn7XjA.png)

- **Contribute to the Project**: Visit the Wing [GitHub repository](https://github.com/winglang/wing) to contribute to its development.
- **Try Wing**: Explore Wing from the [official website](https://www.winglang.io/docs/) and start building your cloud application.
- **Join the Community**: Engage with other users and the development team on [Discord](https://discord.com/invite/d5dzeEewQa) to share insights and get support.

## Conclusion

Winglang is a cloud-oriented programming language designed to simplify cloud development by integrating application code with infrastructure management. Its local development environment, featuring a simulator, allows developers to test and iterate rapidly without deploying to the cloud. Wing also supports multi-cloud deployments, enabling organizations to leverage the best features and pricing from various cloud providers while reducing operational risks.


