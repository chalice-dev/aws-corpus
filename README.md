# Awesome AWS Corpus
> Mission: Create a corpus from the AWS documentation<br/>
> Difficulty: Advanced<br/>
> Awesomeness: Awesome<br/>
> See also: [Awesome AWS Chalice](https://github.com/chalice-dev/awesome-chalice)

Procedure for you to follow along with to download / spider / mirror / crawl a local copy of `docs.aws.amazon.com` from the Internet Archive, and of the [awsdocs GitHub Organization](https://github.com/awsdocs), for applications such as clustering / natural language processing / machine learning or just offline reading. Only downloads `/latest/` and only in English. Remove `html` from the `wayback_machine_downloader` command if you also want images, json, etc. The filtering reduces the number of pages downloaded from 1M+ to just ~100,000 (for both corpora) by carefully eliminating garbage URIs, such as those that include `:80` or `?`. Note that this corpus will have the latest version of the document crawled by the Internet Archive, additionally, it may also have the last published page of documentation that is no longer active on `docs.aws.amazon.com`.

## Setting up Ubuntu Server on a c5a.xlarge instance

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

What's our deepest level of nesting in the directory structure? <code>20</code>:

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

Create a version of the corpus that only has text, with all HTML removed. 

```bash
cd ~
echo '#!/home/ubuntu/anaconda/bin/python
import sys,os;from bs4 import BeautifulSoup
n=sys.argv[1]
f=open(n);
s=BeautifulSoup(f.read(),'lxml');f.close()
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

Remove the HTML. This will take a couple of hours:

```
cd ~
time parallel < html.list
```

Now let's augment this corpus with the [awsdocs GitHub Organization](https://github.com/awsdocs), which has hundreds of repos (`241` as of now) of open source AWS docs full of markdown:

```bash
for i in `seq 1 50`;do curl -s "https://api.github.com/orgs/awsdocs/repos?per_page=20&page=$i" | grep full_name > aws_repos.list
cat aws_repos.list | cut -f2 -d: | sed -r 's/"//;s/ //;s/",//;s,^,https://github.com/,;s/$/\.git/;s/^/git clone /'> aws_docs_git_repos.list
mkdir git_repos
```

```bash
cd ~/git_repos
parallel < ~/aws_docs_git_repos.list
```

How large is this? A healthy `825MB`:

```bash
cd ~
du -sh git_repos
```

Let's delete everything except markdown. New size is `114M`:

```bash
cd ~
find git_repos -type f ! -name '*.md' -delete
```

How much information? `17MB`:

```bash
cd ~
7z a -t7z -m0=lzma -mx=9 -mfb=64 -md=32m -ms=off ~/awsdocs_git_repos.7z ~/git_repos
```

Top level directories:

```
ubuntu@ip-172-31-33-25:~/git_repos$ ls
AWS-Kinesis-Video-Documentation                amazon-managed-blockchain-management-guide        aws-cloudhsm-user-guide                       aws-marketplace-user-guide-for-sellers
GuidesPatternsSolutions                        amazon-migrationhub-user-guide                    aws-cloudtrail-user-guide                     aws-mobile-developer-guide
alexa-for-business-administration-guide        amazon-mq-developer-guide                         aws-codeartifact-user-guide                   aws-net-developer-guide
amazon-api-gateway-developer-guide             amazon-mq-migration-guide                         aws-codebuild-user-guide                      aws-opsworks-user-guide
amazon-application-discovery-user-guide        amazon-mwaa-user-guide                            aws-codecommit-user-guide                     aws-organizations-docs
amazon-appstream2-developer-guide              amazon-personalize-developer-guide                aws-codedeploy-user-guide                     aws-panorama-developer-guide
amazon-athena-user-guide                       amazon-pinpoint-developer-guide                   aws-codepipeline-user-guide                   aws-parallelcluster-user-guide
amazon-aurora-user-guide                       amazon-pinpoint-user-guide                        aws-codestar-user-guide                       aws-personal-health-dashboard
amazon-chime-administration-guide              amazon-polly-developer-guide                      aws-config-developer-guide                    aws-php-developers-guide
amazon-chime-developer-guide                   amazon-quicksight-user-guide                      aws-control-tower-guide                       aws-powershell-user-guide
amazon-chime-user-guide                        amazon-rds-on-vmware-user-guide                   aws-cost-management-user-guide                aws-pricing-calculator-user-guide
amazon-cloud-directory-developer-guide         amazon-rds-user-guide                             aws-cpp-developer-guide                       aws-private-ca-user-guide
amazon-cloudfront-developer-guide              amazon-redshift-developer-guide                   aws-cryptography-overview-developer-guide     aws-proton-administration-guide
amazon-cloudsearch-developer-guide             amazon-redshift-getting-started-guide             aws-cur-user-guide                            aws-proton-user-guide
amazon-cloudwatch-events-user-guide            amazon-redshift-management-guide                  aws-data-exchange-user-guide                  aws-resource-access-manager-user-guide
amazon-cloudwatch-logs-user-guide              amazon-rekognition-custom-labels-developer-guide  aws-data-pipeline-developer-guide             aws-resource-groups-user-guide
amazon-cloudwatch-user-guide                   amazon-rekognition-developer-guide                aws-deep-learning-amis                        aws-robomaker-developer-guide
amazon-codeguru-profiler-user-guide            amazon-route53-docs                               aws-deeplens-user-guide                       aws-ruby-developer-guide
amazon-codeguru-reviewer-user-guide            amazon-s3-developer-guide                         aws-deepracer-developer-guide                 aws-sam-developer-guide
amazon-comprehend-developer-guide              amazon-s3-getting-started-guide                   aws-direct-connect-user-guide                 aws-sct-user-guide
amazon-connect-admin-guide                     amazon-s3-user-guide                              aws-directory-service-admin-guide             aws-sdk-for-javascript-v3
amazon-connect-user-guide                      amazon-s3-userguide                               aws-dms-step-by-step                          aws-secrets-manager-docs
amazon-corretto-11-user-guide                  amazon-sagemaker-developer-guide                  aws-dms-user-guide                            aws-security-hub-partner-guide
amazon-corretto-8-user-guide                   amazon-ses-developer-guide                        aws-doc-sdk-examples                          aws-security-hub-user-guide
amazon-detective-admin-guide                   amazon-sns-developer-guide                        aws-doc-shared-content                        aws-server-migration-service-user-guide
amazon-detective-user-guide                    amazon-sqs-developer-guide                        aws-dynamodb-encryption-docs                  aws-serverlessrepo-developer-guide
amazon-devops-guru-user-guide                  amazon-sumerian-user-guide                        aws-elastic-beanstalk-developer-guide         aws-service-catalog-admin-guide
amazon-dynamodb-developer-guide                amazon-textract-developer-guide                   aws-elemental-mediaconnect-user-guide         aws-single-sign-on-user-guide
amazon-ec2-auto-scaling-user-guide             amazon-traffic-mirroring-guide                    aws-elemental-mediaconvert-user-guide         aws-site-to-site-vpn-user-guide
amazon-ec2-user-guide                          amazon-transcoder-developer-guide                 aws-elemental-medialive-user-guide            aws-snowball-developer-guide
amazon-ec2-user-guide-windows                  amazon-transcribe-developer-guide                 aws-elemental-mediapackage-user-guide         aws-snowball-user-guide
amazon-ecr-public-user-guide                   amazon-translate-developer-guide                  aws-elemental-mediastore-user-guide           aws-step-functions-developer-guide
amazon-ecr-user-guide                          amazon-vpc-network-administrator-guide            aws-elemental-mediatailor-user-guide          aws-storagegateway-user-guide
amazon-ecs-developer-guide                     amazon-vpc-peering-guide                          aws-elemental-speke-drm-docs                  aws-support-user-guide
amazon-efs-user-guide                          amazon-vpc-user-guide                             aws-encryption-sdk-docs                       aws-systems-manager-user-guide
amazon-eks-user-guide                          amazon-workdocs-administration-guide              aws-example-apps                              aws-toolkit-eclipse-user-guide
amazon-elasticache-docs                        amazon-workdocs-dev-guide                         aws-freertos-docs                             aws-toolkit-jetbrains-user-guide
amazon-elasticsearch-service-developer-guide   amazon-workdocs-user-guide                        aws-global-accelerator-dev-guide              aws-toolkit-visual-studio-user-guide
amazon-emr-management-guide                    amazon-workmail-administration-guide              aws-glue-developer-guide                      aws-toolkit-vs-code-user-guide
amazon-emr-release-guide                       amazon-workmail-user-guide                        aws-glue-user-guide                           aws-tools-ado-vsts-user-guide
amazon-eventbridge-user-guide                  amazon-workspaces-administration-guide            aws-go-developer-guide                        aws-transit-gateway-guide
amazon-forecast-developer-guide                amazon-workspaces-user-guide                      aws-greengrass-developer-guide                aws-unity-developer-guide
amazon-fsx-lustre-user-guide                   application-auto-scaling-user-guide               aws-ios-developer-guide                       aws-waf-and-shield-advanced-developer-guide
amazon-fsx-windows-user-guide                  aws-1-click-developer-guide                       aws-iot-analytics-user-guide                  aws-xamarin-developer-guide
amazon-gamelift-developer-guide                aws-amplify-console-user-guide                    aws-iot-docs                                  aws-xray-developer-guide
amazon-glacier-developer-guide                 aws-android-developer-guide                       aws-iot-events-developer-guide                ec2-image-builder-user-guide
amazon-guardduty-user-guide                    aws-app-mesh-user-guide                           aws-iot-greengrass-v2-developer-guide         elastic-beanstalk-samples
amazon-inspector-user-guide                    aws-artifact-user-guide                           aws-iot-sitewise-monitor-application-guide    elb-application-load-balancers-user-guide
amazon-kendra-developer-guide                  aws-auto-scaling-user-guide                       aws-iot-sitewise-user-guide                   elb-classic-load-balancers-user-guide
amazon-kinesis-data-analytics-developer-guide  aws-batch-user-guide                              aws-iot-things-graph-user-guide               elb-gateway-load-balancers-user-guide
amazon-kinesis-data-firehose-developer-guide   aws-blockchain-templates-developer-guide          aws-iotanalytics-developer-guide              elb-network-load-balancers-user-guide
amazon-kinesis-data-streams-developer-guide    aws-cdk-guide                                     aws-java-developer-guide                      elb-user-guide
amazon-lex-developer-guide                     aws-certificate-user-guide                        aws-java-developer-guide-v2                   emp-user-guide
amazon-lightsail-developer-guide               aws-chatbot-admin-guide                           aws-javascript-developer-guide-v2             iam-user-guide
amazon-lookout-for-vision-developer-guide      aws-cli-user-guide                                aws-kinesis-video-webrtc-documentation        macie-user-guide
amazon-lookoutmetrics-developer-guide          aws-client-vpn-administrator-guide                aws-kms-developer-guide                       nice-dcv-admin-guide
amazon-lumberyard-getting-started-guide        aws-client-vpn-user-guide                         aws-lambda-developer-guide                    nice-dcv-user-guide
amazon-lumberyard-release-notes                aws-cloud-map-developer-guide                     aws-launch-wizard-user-guide                  porting-assistant-user-guide
amazon-lumberyard-user-guide                   aws-cloud9-user-guide                             aws-management-portal-for-vcenter-user-guide  service-quotas-docs
amazon-machinelearning-developer-guide         aws-cloudformation-user-guide                     aws-marketplace-user-guide-for-buyers         vm-import-export-user-guide
```


