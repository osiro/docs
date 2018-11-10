---
title: PubSub
---

# PubSub

PubSub provides connectivity with cloud-based message-oriented middleware. You can use PubSub to pass messages between your app instances and your app's backend creating real-time interactive experiences.

PubSub is available with **AWS IoT**. 

When using AWS IoT your PubSub HTTP requests are automatically signed when sending your messages.
{: .callout .callout--info}

## Installation and Configuration

### AWS IoT

When used with `AWSIoTDataManager`, PubSub is capable of signing request according to [Signature Version 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html). 

The `Podfile` that you configure to install the AWS Mobile SDK must contain the `AWSIoT` pod:

```ruby
    platform :ios, '9.0'

    target :'YOUR-APP-NAME' do
      use_frameworks!

        pod  'AWSIoT', '~> 2.7.0'
        # other pods

    end
```

Run `pod install --repo-update` before you continue.

To use in your app, import the following:

```swift
import AWSIoT
```

Define your unique client ID and endpoint (incl. region) in your configuration:

```swift
// Initialize the AWSIoTDataManager with the configuration
let iotEndPoint = AWSEndpoint(
    urlString: "wss://xxxxxxxxxxxxx.iot.<YOUR-AWS-REGION>.amazonaws.com/mqtt")
let iotDataConfiguration = AWSServiceConfiguration(
    region: AWSRegionType.<YOUR-AWS-REGION>,
    endpoint: iotEndPoint,
    credentialsProvider: AWSMobileClient.sharedInstance()
)

AWSIoTDataManager.register(with: iotDataConfiguration!, forKey: ASWIoTDataManager)
AWSIoTDataManager iotDataManager = AWSIoTDataManager(forKey: ASWIoTDataManager)                                               
```

**Create IAM policies for AWS IoT**

To use PubSub with AWS IoT, you will need to create the necessary IAM policies in the AWS IoT Console, and attach them to your Amazon Cognito Identity. 

Go to IoT Core and choose *Secure* from the left navigation pane. Then navigate to *Create Policy*. The following `myIOTPolicy` policy will allow full access to all the topics.

![Alt text]({%if jekyll.environment == 'production'%}{{site.amplify.docs_baseurl}}{%endif%}/js/images/iot_attach_policy.png?raw=true "Title")


**Attach your policy to your Amazon Cognito Identity**

To attach the policy to your *Cognito Identity*, begin by retrieving the `Cognito Identity Id` from `AWSMobileClient`.

```swift
AWSMobileClient.sharedInstance().getIdentityId();
```

Then, you need to attach the `myIOTPolicy` policy to the user's *Cognito Identity Id* with the following [AWS CLI](https://aws.amazon.com/cli/) command:

```bash
aws iot attach-principal-policy --policy-name 'myIOTPolicy' --principal '<YOUR_COGNITO_IDENTITY_ID>'
```

## Working with the API

### Establish Connection

Before you can subscribe to a topic, you need to establish a connection as follows:

```swift
iotDataManager.connect(
    withClientId: "<YOUR_CLIENT_ID>",
    cleanSession: true,
    certificateId: "<YOUR_CERTIFICATE_ID>") { (status) in
         print("Connection Status: \(status.rawValue)")
}
```

### Subscribe to a topic

In order to start receiving messages from your provider, you need to subscribe to a topic as follows;

```swift
iotDataManager.subscribe(
    toTopic: "myTopic",
    qoS: .messageDeliveryAttemptedAtMostOnce, /* Quality of Service */
    messageCallback: {
        (payload) ->Void in
        let stringValue = NSString(data: payload, encoding: String.Encoding.utf8.rawValue)!

        print("Message received: \(stringValue)")
} )
```

### Subscribe to multiple topics

To subscribe for multiple topics, just call `subscribe()` for each topic you wish to subscribe. 

### Publish to a topic

To send a message to a topic, use `publishString()` method with your topic name and the message:

```swift
iotDataManager.publishString(
    "Hello to all subscribers!",
    onTopic: "myTopic", 
    qoS:.messageDeliveryAttemptedAtMostOnce)
```

### Unsubscribe from a topic

To stop receiving messages from a topic, you can use `unsubscribeTopic()` method:

```swift
iotDataManager.unsubscribeTopic("myTopic")

// You will no longer get messages for "myTopic"
```

### Close Connection

In order to disconnect, you need to close the connection as follows:

```swift
iotDataManager.disconnect()
```

### API Reference

For the complete API documentation for AWS IoT, visit our [API reference](https://aws-amplify.github.io/aws-sdk-ios/docs/reference/Classes/AWSIoTDataManager.html)
{: .callout .callout--info}