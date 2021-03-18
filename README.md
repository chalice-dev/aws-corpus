# Awesome AWS Corpus
> Mission: Create a corpus from the AWS documentation<br/>
> Difficulty: Advanced<br/>
> Awesomeness: Awesome

One-liner to download / spider / mirror / crawl a local copy of `docs.aws.amazon.com` from the Internet Archive, for applications such as clustering / natural language processing / machine learning or just offline reading. Only downloads `/latest/` and only in English. Remove `html` from the command if you also want images, json, etc. The filtering reduces the number of pages downloaded from 1M+ to just ~85,000 by carefully eliminating garbage URIs, such as those that include `:80` or `?`. Note that this corpus will have the latest version of the document crawled by the Internet Archive, additionally, it may also have the last published page of documentation that is no longer active on `docs.aws.amazon.com`.

## Installation

```bash
#!/bin/bash
# For Ubuntu
cd ~
sudo apt -y update
sudo apt-get -y install ruby rdfind p7zip-full parallel
sudo gem install wayback_machine_downloader
wget https://repo.anaconda.com/archive/Anaconda3-2020.11-Linux-x86_64.sh -O ~/anaconda.sh
bash ~/anaconda.sh -b -p $HOME/anaconda
echo 'export PATH=$PATH:~/anaconda/bin' >> ~/.bashrc
source ~/.bashrc
conda install spacy
```

## Spidering / Crawling / Mirroring

You should have the site by the same time the next day. Run this in `~`.

`time wayback_machine_downloader --only '/^https:\/\/docs\.aws\.amazon\.com\/.+\/latest\/.+\.html$/' --exclude '/docs\.aws\.amazon\.com\/[a-z]+_[a-z]+\//' --concurrency 5 https://docs.aws.amazon.com`

### Processing the corpus

You can remove any duplicate files (identical contents) with `rdfind`:

```bash
cd ~
rdfind -deleteduplicates true websites
```

Remove any files or directories with '?' that made it in:

```bash
cd ~
find . | grep '?' | xargs -L1 rm -fr
find . | grep '?' | xargs -d "\n" -I {} rm -fr '{}'
```

Check how many pages of documentation we ended up with, in this case, `86,570`:

```bash
cd ~
find . | grep html | wc -l
```

What's our deepest level of nesting in the direction structure? <code>20</code>

```bash
cd ~
find . -type d | sed 's|[^/]||g' | sort | tail -n1 | wc -c
```

Compute an estimate of the amount of information in the corpus (and create a compressed archive), in this case `677MB`:

```bash
cd ~
7z a -t7z -m0=lzma -mx=9 -mfb=64 -md=32m -ms=off ~/docs.aws.amazon.com.7z ~/websites/docs.aws.amazon.com/
```

Inspect our top-level directory structure:

```
ubuntu@ip-172-31-33-25:~$ ls websites/docs.aws.amazon.com/
ARG                                          ElasticLoadBalancing   aws-iq                       cognitosync                 elastic-inference              inspector                      mediapackage              rdsdataservice                systems-manager
AWSAndroidSDK                                ElasticMapReduce       aws-mobile                   comprehend                  elasticbeanstalk               iot                            mediapackage-vod          redshift                      textract
AWSCloudFormation                            IAM                    aws-technical-content        compute-optimizer           elasticloadbalancing           iot-1-click                    mediastore                rekognition                   thingsgraph
AWSEC2                                       Monitron               awsaccountbilling            config                      elasticsearch-service          iot-sitewise                   mediatailor               resourcegroupstagging         timestream
AWSECommerceService                          Route53                awscloudtrail                connect                     elastictranscoder              iotanalytics                   migrationhub              rfdk                          toolkit-for-jetbrains
AWSGettingStartedContinuousDeliveryPipeline  STS                    awsconsolehelpdocs           connect-participant         elemental-appliances-software  iotevents                      migrationhub-home-region  robomaker                     toolkit-for-visual-studio
AWSImportExport                              SchemaConversionTool   awssupport                   controltower                elemental-cf2                  ivs                            mobileanalytics           sagemaker                     toolkit-for-vscode
AWSJavaSDK                                   a4b                    batch                        corretto                    elemental-cl3                  kendra                         msk                       sap                           transcribe
AWSJavaScriptSDK                             access-analyzer        blockchain-templates         credref                     elemental-delta                keyspaces                      mwaa                      savingsplans                  transfer
AWSMechTurk                                  acm                    braket                       crypto                      elemental-live                 kinesis                        mxnet                     sdk-for-java                  translate
AWSRubySDK                                   acm-pca                cdk                          cur                         elemental-server               kinesis-agent-windows          neptune                   sdk-for-net                   vm-import
AWSSdkDocsJava                               amazon-mq              chatbot                      data-exchange               emp                            kinesisanalytics               network-firewall          sdk-for-ruby                  vpc
AWSSdkDocsNET                                amazondynamodb         chime                        databrew                    emr                            kinesisvideostreams            networkmanager            sdkfornet                     vpn
AWSSdkDocsRuby                               amazonglacier          cli                          datapipeline                enclaves                       kinesisvideostreams-webrtc-dg  opsworks                  sdkfornet1                    vsts
AWSSimpleQueueService                        amazonswf              cloud-map                    datasync                    encryption-sdk                 kms                            opsworks-cm               secretsmanager                waf
AWSToolkitEclipse                            amp                    cloud9                       dcv                         eventbridge                    lake-formation                 organizations             securityhub                   wam
AWSUnitySDK                                  amplify                clouddirectory               deep-learning-containers    firehose                       lambda                         outposts                  server-migration-service      wavelength
AWSiOSSDK                                    apigateway             cloudformation-cli           deepcomposer                forecast                       launchwizard                   parallelcluster           serverless-application-model  wellarchitected
AlexaTopSites                                apigatewayv2           cloudfront                   deeplens                    frauddetector                  lex                            performance-insights      serverlessrepo                whitepapers
AlexaWebInfoService                          app-mesh               cloudhsm                     deepracer                   freertos                       lexv2                          personalize               service-authorization         workdocs
AmazonCloudFront                             app2container          cloudsearch                  detective                   freertos-kernel                license-manager                pinpoint                  servicecatalog                worklink
AmazonCloudWatch                             appconfig              cloudshell                   devicefarm                  fsx                            location                       pinpoint-email            servicequotas                 workmail
AmazonCloudWatchEvents                       appflow                code-samples                 directconnect               gamelift                       location-places                pinpoint-sms-voice        ses                           workspaces
AmazonCloudWatchLogs                         appinsights            codeartifact                 directoryservice            general                        lumberyard                     polly                     signer                        xray
AmazonDevPay                                 appintegrations        codebuild                    dlami                       gettingstarted                 machine-learning               portingassistant          silk                          xray-sdk-for-dotnet
AmazonECR                                    application-discovery  codecommit                   dlm                         global-accelerator             macie                          powershell                singlesignon                  xray-sdk-for-java
AmazonECS                                    appstream              codedeploy                   dms                         glue                           managed-blockchain             prescriptive-guidance     snowball                      xray-sdk-for-nodejs
AmazonElastiCache                            appstream2             codeguru                     documentdb                  govcloud-us                    marketplace                    pricing-calculator        sns                           xray-sdk-for-python
AmazonRDS                                    appsync                codepipeline                 dtconsole                   grafana                        marketplace-catalog            prometheus                solutions
AmazonS3                                     artifact               codestar                     dynamodb-encryption-client  greengrass                     marketplaceentitlement         proton                    speke
AmazonSimpleDB                               athena                 codestar-connections         ebs                         ground-station                 marketplacemetering            qldb                      ssm
AmazonSynthetics                             audit-manager          codestar-notifications       ec2-instance-connect        guardduty                      mcs                            quicksight                step-functions
AmazonVPC                                    autoscaling            cognito                      eclipse-toolkit             health                         mediaconnect                   quickstart                storagegateway
ApplicationAutoScaling                       aws-backup             cognito-user-identity-pools  efs                         honeycode                      mediaconvert                   quickstarts               streams
AutoScaling                                  aws-cost-management    cognitoidentity              eks                         imagebuilder                   medialive                      ram                       sumerian
```

Create a version of the corpus that only has text, with all HTML removed:

```bash
cd ~
echo '#!/home/ubuntu/anaconda/bin/python
import sys,os;from bs4 import BeautifulSoup
n=sys.argv[1]
f=open(n);
s=BeautifulSoup(f.read());f.close()
t=s.get_text()
d=n.replace("websites/","websites/txt/")
p=d.split("/")
o="/".join(p[:-1])
os.makedirs(o,exist_ok=True)
f=open(d,"w");f.write(t);f.close()' > ~/strip_text
chmod +x strip_text
```

```bash
cd ~
find websites/ -type f -name '*.html' > html.list
sed -r -i 's/^(.*)/~\/strip_text "\1"/' html.list
```

```

```
