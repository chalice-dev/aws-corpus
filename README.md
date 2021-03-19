# AWS Corpus
> Mission: Create a corpus from the AWS documentation<br/>
> Difficulty: Advanced<br/>
> See also: [Awesome AWS Chalice](https://github.com/chalice-dev/awesome-chalice)

Procedure for you to follow along with to download / spider / mirror / crawl a local copy of `docs.aws.amazon.com` from the Internet Archive, and of the [awsdocs GitHub Organization](https://github.com/awsdocs), for applications such as clustering / natural language processing / machine learning or just offline reading. Only downloads `/latest/` and only in English. Remove `html` from the `wayback_machine_downloader` command if you also want images, json, etc. The filtering reduces the number of pages downloaded from 1M+ to just ~100,000 (for both corpora) by carefully eliminating garbage URIs, such as those that include `:80` or `?`. Note that this corpus will have the latest version of the document crawled by the Internet Archive, additionally, it may also have the last published page of documentation that is no longer active on `docs.aws.amazon.com`.

## Setting up Ubuntu Server on a c5a.xlarge and c5.24xlarge	instance

Most code was run on a `c5a.xlarge`, however, for long runs with `parallel`, you can temporarily switch over to a `c5.24xlarge` (96 CPUs) to finish in no time.

```bash
#!/bin/bash
cd ~
sudo apt -y update
sudo apt-get -y install ruby rdfind p7zip-full parallel lrzip
sudo gem install wayback_machine_downloader
wget https://repo.anaconda.com/archive/Anaconda3-2020.11-Linux-x86_64.sh -O ~/anaconda.sh
bash ~/anaconda.sh -b -p $HOME/anaconda
echo 'export PATH=$PATH:~/anaconda/bin' >> ~/.bashrc
source ~/.bashrc
conda install -c conda-forge spacy
conda install -c conda-forge markdown
conda install -c conda-forge nltk
python -m spacy download en_core_web_sm
```

## Spidering / Crawling / Mirroring

You should have the site by the same time the next day. Run this in `~`.

```bash
cd ~
time wayback_machine_downloader --only '/^https:\/\/docs\.aws\.amazon\.com\/.+\/latest\/.+\.html$/' \
                                --exclude '/docs\.aws\.amazon\.com\/[a-z]+_[a-z]+\//' \
                                --concurrency 5 \
                                https://docs.aws.amazon.com`
```

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

How large is this? A healthy `825MB` and `10,612` markdown files:

```bash
cd ~
du -sh git_repos
find git_repos -type f -name "*.md" | wc -l
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

Let's [strip the markdown](https://stackoverflow.com/questions/761824/python-how-to-convert-markdown-formatted-text-to-text) so we have just text. Save this as `strip_markdown` in `~` and run `chmod +x strip_markdown` on it.

```python
#!/home/ubuntu/anaconda/bin/anaconda
from markdown import Markdown
from io import StringIO
import sys, os


def unmark_element(element, stream=None):
    if stream is None:
        stream = StringIO()
    if element.text:
        stream.write(element.text)
    for sub in element:
        unmark_element(sub, stream)
    if element.tail:
        stream.write(element.tail)
    return stream.getvalue()


# patching Markdown
Markdown.output_formats["plain"] = unmark_element
__md = Markdown(output_format="plain")
__md.stripTopLevelTags = False

def unmark(text):
    return __md.convert(text)

path = sys.argv[1]

f = open(path);txt = f.read();f.close()

txt_clean = unmark(txt)

path_parts = path.split("/")

output_dir = "/".join(path_parts[:-1])
output_dir = output_dir.replace("git_repos", "git_repos_txt")

os.makedirs(output_dir, exist_ok=True)

f = open(output_dir + "/" + path_parts[-1], "w")
f.write(txt_clean)
f.close()

```

Create a list of commands to run on the the markdown files in parallel:

```
cd ~
find git_repos -type f -name "*.md" > md.list
sed -ri 's/^/~\/strip_markdown /' md.list
```

Strip the markdown:

```
cd ~
time parallel < md.list
```

It turns out that only `74` of the `241` `awsdocs` repos contained markdown.

With all html and markdown removed, `docs.aws.amazon.com` is now `2GB` uncompressed and `298MB` compressed, and `awsdocs` is now `75MB` uncompressed and `40MB` compressed. 

Let's combine them into one corpus, and estimate their [mutual information](https://en.wikipedia.org/wiki/Mutual_information).

```bash
cd ~
mkdir ~/aws_corpus
cp -r websites/txt/docs.aws.amazon.com ~/aws_corpus/docs.aws.amazon.com
mkdir aws_corpus/awsdocs
cp -r git_repos_txt/* ~/aws_corpus/awsdocs/
7z a -t7z -m0=lzma -mx=9 -mfb=64 -md=32m -ms=off ~/aws_corpus.7z ~/aws_corpus
```

The _expected size_ of the compressed version of the parallel corpus is `311495596 + 15024850 = 326520446` bytes, which is the size of each compressed corpus alone. Any improvement on this when they are compressed together must be due to their mutual information. The compressed size of the parallel corpus is `326429896` bytes, which is only `90,550` bytes or `90.5KB` smaller than expected 

This must be partly due to using the wrong compression algorithm, as 7zip can't compress streams. Indeed, `aws_corpus.tar.gz` is `254MB` compared to `aws_corpus.7z` (above) which is `312MB`.

Lets see if we can use NLP to bring these corpora into closer alignment by running the same [spaCy text preprocessor](https://gist.github.com/omri374/ec1c243a5a94a657dae40078d47977b6) on both corpora, including stop word removal, lemmatization, etc. 

```bash
cd ~
wget https://gist.githubusercontent.com/brianmingus2/54645d9a551ec8d96e6cd4a8c7e2abaf/raw/c09de34d804c3b05fa61f65d8dd146b83438c663/spacy_preprocessor.py -O spacy_pre
chmod + x spacy_pre
find aws_corpus/ -type f -name "*.html" > aws_corpus.list
find aws_corpus/ -type f -name "*.md" >> aws_corpus.list
sed -ri 's/^/~\/spacy_pre //' aws_corpus.list
```

Run the preprocessor:

```bash
cd ~
parallel < aws_corpus.list
```

Compress the parallel corpus with `lrztar`:

```bash
cd ~
lrztar aws_corpus_spaCy
```

Now, the preprocessed `docs.aws.amazon.com` is `1.3GB` uncompressed, and `28MB` when compressed with `lrztar`, a compression ratio of `41.6`, whereas `awsdocs` is `62MB` uncompressed and `4.6M` compressed, a compression ratio of `9.3`. The expected size of the compressed parallel corpus is `33451720` bytes, whereas the actual size is `31810052` bytes, an improvement of `1641668` bytes or `1.6MB` over our expectation.

Let's take all the text for each corpus, and put them into one large file per corpus, one large file overall, and compare compression ratios:

```bash
find awsdocs -type f -exec cat {} + > awsdocs.txt
find docs.aws.amazon.com -type f -exec cat {} + > docsaws.txt
cat awsdocs.txt docsaws.txt > docs.txt 
lrzip awsdocs.txt
lrzip docsaws.txt
lrzip docs.txt
```

Now, `docsaws` is `25MB`, `awsdocs` is `4.3MB` and `docs` is, again, `28MB`, and we improved compression over our expectation by an additional `64KB`. 

Up to now, the words in our documents were in the same order they appeared in the original document. Let's try putting each word in `docs` on one line, sorting the entire thing, and then compressing. This shows we have `68,856,518` words in our corpus:

```bash
cat docs.txt | tr " " "\n" > words.list
cat docs.txt | tr " " "\n" | grep . > words.list
lrzip words_sorted.list
```

The compressed list of sorted words is only `1.6MB`, demonstrating that it takes `26.4MB` of space to represent the order of words in our corpus. 

Let's test our compressor by making every word in the sorted corpus unique, prepending it with a count of the number of times it occurs, and sorting again. In theory, the compressor could use this information: 

```bash
cat words_sorted.list | uniq -c > words_sorted_uniq.list
cat words_sorted.list | uniq -c | sort -n --parallel=96 | tac > words_sorted_uniq.list
lrzip words_sorted_uniq.list
```

Our corpus is now only `1.5MB`. What if we remove the counts of each word, resulting in just one word per line? Our corpus shrinks to just `1.1MB`. There are `316,164` "words" in the file, which is only 50% larger than in English.

```bash
cat words_sorted.list | uniq | sort -n --parallel=96 | tac > words_sorted_uniq_nocount.list
lrzip words_sorted_uniq_nocount.list
```

We can also view `words_sorted_uniq.list` to see the most frequent words in our corpus, and how often they occur. No surprises here:

```
2458894 aws
 768456 string
 739287 property
 633644 amazon
 601828 return
 583947 cdk
 546867 value
 496027 object
 459157 request
 413411 class
 403076 public
 401986 type
 395220 use
 384263 method
 379585 var
 378868 specify
 ```

Let's create one corpus per product, in each of `awsdocs` and `docsaws`, so that we can create product vectors. This results in `370` corpora:

```bash
cd ~
cd aws_corpus_spaCy
cd awsdocs
for product in $(ls);do find $product -type f -exec cat {} + > $product.corpus;done
cd ../docs.aws.amazon.com
for product in $(ls);do find $product -type f -exec cat {} + > $product.corpus;done
cd ..
mkdir products
find . -type f -name "*.corpus" -exec mv -t products {} +
```


