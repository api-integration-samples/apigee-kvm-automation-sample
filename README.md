# Apigee KVM Automation Sample
This sample shows how to use automation to manage a simple KVM (KeyValueMap) in Apigee X to store and retrieve configuration or secret values. This sample can be run very easily in [Google Cloud Shell](https://shell.cloud.google.com), and use a free Apigee X instance for testing.

Resources created in this sample:
- An Apigee KVM is created for storing secrets.
- A KVM entry is created to store a string value.
- An Apigee proxy is deployed that retrieves that KVM value and returns it in an API call.

## Prerequisites
To run this sample, you need a **Google Cloud project** with any version of **Apigee X** provisioned, and at least the **API Admin (V2)** user role assigned to read/write KVMs. You will also need the **gcloud** tool installed, which it is by default in [Google Cloud Shell](https://shell.cloud.google.com)

## Instructions

### Step 1: Open Cloud Shell or your Linux shell of choice
Open [Google Cloud Shell](https://shell.cloud.google.com) for easy Google Cloud automation in a free & managed Linux shell environment, or another shell of choice.

### Step 2: Install the apigeecli tool
[**apigeecli**](https://github.com/apigee/apigeecli) is a tool for automating any operation in Apigee X. To begin, make sure that the tool is installed by running this script.

```sh
# install apigeecli
curl -L https://raw.githubusercontent.com/apigee/apigeecli/main/downloadLatest.sh | sh -

# add to .bashrc path
echo $'export PATH=$PATH:$HOME/.apigeecli/bin' >> ~/.bashrc
source ~/.bashrc
```

### Step 3: Set environment variables
```sh
# copy the 1.env.sh file to edit locally
cp 1.env.sh 1.env.local.sh
# edit and save your Apigee X Project Id, Region & Environment
nano 1.env.local.sh
# source results
source 1.env.local.sh
```

### Step 4: Create a KVM
This command will create a KVM called `test-kvm` in your Apigee project/org. Here are the docs for the [apigeecli command kvms create](https://github.com/apigee/apigeecli/blob/main/docs/apigeecli_kvms_create.md).

```sh
# create a KVM using the apigeecli tool
apigeecli kvms create -e $APIGEE_ENV -n test-kvm -o $PROJECT_ID -t $(gcloud auth print-access-token)
```

### Step 5: Create a KVM entry
Here we are going to create a key-value entry called **test-entry** with the value **hello kvm!**, which can later be easily accessed and used by an API proxy.

```sh
# create a KVM entry using the apigeecli tool
apigeecli kvms entries create -m test-kvm -k test-entry -l "hello kvm!" -e $APIGEE_ENV -o $PROJECT_ID -t $(gcloud auth print-access-token)
```

### Step 6: Deploy an API proxy that uses the KVM entry

```sh
# change directory to our test proxy dir
cd src/main/apigee/apiproxies/KvmTest-v1
# create an Apigee bundle
apigeecli apis create bundle -f apiproxy --name KvmTest-v1 -o $PROJECT_ID -t $(gcloud auth print-access-token)
# deploy the proxy to an Apigee environment
apigeecli apis deploy -n KvmTest-v1 -o $PROJECT_ID -e $APIGEE_ENV -t $(gcloud auth print-access-token) --ovr
# change back to root dir
cd ../../../../..
```

### Step 7: Test the proxy

```sh
# get the environment group hostname
HOST=$(apigeecli envgroups get -n $APIGEE_ENVGROUP -o $PROJECT_ID -t $(gcloud auth print-access-token) | jq --raw-output '.hostnames[0]')
# do a test call to our new API proxy
curl -i "https://$HOST/v1/kvmtest"
# you should get 'hello kvm!' back. congrats, you've finished!
```