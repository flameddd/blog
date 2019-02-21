## docker、gitlab-ci、k8s隨手看


## envsubst
利用 Linux 指令 envsubst 產生設定檔  
寫 Dockerfile 時會要將`變數`設定檔放進 Docker 內的需求。  

例如以下是設定檔樣板範例：
```sh
My name is ${USER}
My home path is ${HOME}
```

而 envsubst 所做的事就是將設定檔樣板(template)內的變數用相對應的環境變數的值取代掉。
```
$ envsubst < myconfig.conf
```

輸出結果
```
My name is foo
My home path is /home/foo
```

要生成設定檔，就只要再加上重導向輸出到一個檔案即可，例如：
```sh
$ envsubst < myconfig.conf > config.conf
```

[https://github.com/a8m/envsubst](https://github.com/a8m/envsubst)  
`${var}`	Value of var (same as `$var`)  
`$$var`	Escape expressions. Result will be `$var`.



## GitLab CI/CD Variables
這邊的 `$CI_PROJECT_PATH` 是 gitlab ci 的變數  
https://docs.gitlab.com/ee/ci/variables/

```yml
variables:
  IMAGE_NAME: $DOCKER_REGISTRY_HOST/$CI_PROJECT_PATH
```
`CI_PROJECT_PATH`	The namespace with project name  
`$CI_COMMIT_SHA` The commit revision for which project is built 也是環境變數  
`CI_COMMIT_REF_NAME` The branch or tag name for which project is built  

當然有所謂的讀取優先級：  
The order of precedence for variables is (from highest to lowest):  
  1. Trigger variables or scheduled pipeline variables.
  2. Project-level variables or protected variables.
  3. Group-level variables or protected variables.
  4. YAML-defined job-level variables.
  5. YAML-defined global variables.
  6. Deployment variables.
  7. Predefined environment variables.

## only && except
當 job 被 create 時，會判斷這兩個變數。  
- https://docs.gitlab.com/ee/ci/yaml/#only-and-except-simplified

1. `only` defines the **names of branches** and **tags** for which the job will run.
2. `except` defines the **names of branches** and **tags** for which the job will **not run**.

```yml
lint:
  <<: *nodejs-pre-do
  stage: lint
  script: yarn lint
  except:
    - develop
    - /^release\//
    - /^v\d+\.\d+\.\d+$/
```

- develop branch(or tag)不跑
- release 字首的 branch 不跑
- v0.0.0 版本號字結尾的不跑

```yml
.image-env-develop: &image-env-develop
  variables:
    IMAGE_TAG: develop
  only:
    - develop
    - /dockerize/
```

這邊的 `/dockerize/` 是什麼意思？ => 正規表達式而已。我想太多了，就是 match 現在這個開發中的 branch name `feature/dockerize-and-k8s`

## kubectl
`Minimal docker image containing bash, kubectl, ca-certificates & gettext. nothing more, nothing less` <＝ 這是什麼意思？ 我想更清楚一點。

- 那一般來說的 image 跟這個 mini 的差多少 size 呢？
- 一般是用哪個 image 呢？

- `ca-certificates` 查到都是為了 `alpine` 處理 **CA certificates** 的問題。

看到有人寫這樣的說法
```
[問題一] x509: failed to load system roots and no roots provided
成因，alpine 裏面缺失 root ca，導致訪問第三方 ssl 的服務時，無法處理。安裝對應的 ca 證書即可。
RUN apk upgrade && apk add --no-cache ca-certificates
```

為什麼要裝 `dumb-init` 呢？  
```dockerfile
FROM bash:4.4.23 as base
RUN apk --no-cache add gettext ca-certificates openssl \
    && wget https://github.com/Yelp/dumb-init/releases/download/v1.2.0/dumb-init_1.2.0_amd64 -O /usr/local/bin/dumb-init \
    && wget https://storage.googleapis.com/kubernetes-release/release/v1.13.0/bin/linux/amd64/kubectl -O /usr/local/bin/kubectl \
    && chmod a+x /usr/local/bin/kubectl /usr/local/bin/dumb-init \
    && apk --no-cache del ca-certificates openssl
ENTRYPOINT ["/usr/local/bin/dumb-init","--","/usr/local/bin/docker-entrypoint.sh"]
CMD ["bash"]

FROM base as with-ca
RUN apk --no-cache add ca-certificates
```

我還是不懂這邊的原理 `ca-certificates`  
為什麼最後又執行一次 `RUN apk --no-cache add ca-certificates`  

```yml
# gitlab-ci.yml
variables:
  IMAGE: $DOCKER_REGISTRY_HOST/$CI_PROJECT_PATH

stages:
  - image-release

release:image:
  image: docker:stable
  services:
    - docker:dind
  stage: image-release
  cache: {}
  before_script:
    - docker info
    - docker login -u $DOCKER_REGISTRY_USER -p $DOCKER_REGISTRY_PASSWORD $DOCKER_REGISTRY_HOST
  script:
    - export IMAGE_TAG=${CI_COMMIT_REF_NAME//v/}
    - docker build --target base --label "commit=$CI_COMMIT_SHA" -t $:$IMAGE_TAG .
    - docker build --target with-ca --label "commit=$CI_COMMIT_SHA" -t $IMAGE:$IMAGE_TAG-ca .
    - docker tag $IMAGE:$IMAGE_TAG $IMAGE:latest
    - docker push $IMAGE:$IMAGE_TAG
    - docker push $IMAGE:latest
    - docker push $IMAGE:$IMAGE_TAG-ca
  only:
    - /^v\d+\.\d+\.\d+$/
  tags:
    - dockerize
```

`dind` 指的是 docker in docker。在 docker 裡片跑 docker，看起來蠻多人用來 build image 之類（幫忙 CI CD）的。也是有一些缺點在，但這個案例上，就算是很方便。
```yaml
services:
  - docker:dind
```

`--target` 是	Set the target build stage to build。 看不懂，什麼是 build stage？  
這邊看起來是指 dockerfile 的 `base` 跟 `with-ca`  
`$IMAGE_TAG-ca` 不懂這是什麼意思？  
`tags` 呢？  

`tags` Defines a list of tags which are used to select Runner，這個是在指名哪個 `runner` 跑 job

```yaml
- echo -n $KUBE_CONFIG | base64 -d > $HOME/.kube/config
```
`-n` flag Do not print the trailing newline character.  輸出時，不會帶有 newline char

`|`	管線 (pipe)：分隔兩個管線命令的界定(後兩節介紹)  
`ls -al /etc | less`  
如此一來，使用 ls 指令輸出後的內容，就能夠被 less 讀取，並且利用 less 的功能，我們就能夠前後翻動相關的資訊了！很方便是吧？我們就來瞭解一下這個管線命令  

`base64 -d` 輸出 base64 而且 decode  
`> $HOME/.kube/config` 的 `>` 是 bash 的輸出檔案指令。  

`- kubectl config set-context local --namespace=$K8S_NAMESPACE`  
kubectl config set-context 在kubeconfig配置文件中設置一個環境項。  
在kubeconfig配置文件中設置一個環境項。 如果指定了一個已存在的名字，將合並新字段並覆蓋舊字段。  
kubectl config set-context NAME [--cluster=cluster_nickname] [--user=user_nickname] [--namespace=namespace]  
設置gce環境項中的user字段，不影響其他字段。  
`kubectl config set-context gce --user=cluster-admin`  

`--namespace=""`: 設置kuebconfig配置文件中環境選項中的命名空間。  

這邊為什麼叫做 `local` ?? 不懂

`sed -e`: 
- `-e`	執行 sed 的 script 語法	如沒使用〝-f〞選項此為預設選項
- `-f`	選用外部的 script 檔來執行

sed 最基本的命令為搜尋樣板並取代〝s〞,基本用法為 `s/樣板(PATTERN)/取代(REPLACEMENT)/`  

- k8s/mobile-sns.yml.template
- k8s/ingress.yml.template
有兩個 yaml 檔案？我猜一個是部署 pod。

### 開始進入 k8s 了～
apiVersion 不一樣？為什麼這樣選擇？查看看 gitlab
- apiVersion: extensions/v1beta1
- apiVersion: v1

為什麼 `Ingress` 要特別用 `extensions/v1beta1` ？ ＝＞ 官方文件就這樣寫！

`metadata` - 幫助識別對象唯一性的數據，包括一個 name 字符串、UID 和可選的 namespace

## Labels (k8s)
`Labels` 就是一對具有辨識度的key/value。以下面為例：  
```
"release" : "stable"，"release" : "qa"
"enviroment": "dev"，"enviroment": "production"
"tier": "backend", "tier": "frontend"
"department": "enginnerting", "department": "marketing", "department": "finance"
```

若 k8s Cluster中的 Pod 物件帶有標籤，由上面幾個 labels 可以清楚知道，他們的功用是什麼、層級是什麼，他們服務的對象是哪些部門的人。Labels 有以下幾個特點：  

- 每個物件可以同時擁有許多個labels(multiple labels)
- 可以透過 Selector，幫我們縮小要尋找的物件。
- 目前 API 提供不再只是一個 key對應一個value(Equality-based requirement)的關係，我們也可以使用 matchExpressions 來設定更有彈性的Labels。

### 常見的Label
一般來說，我們會給一個Pod（或其他對象）定義多個Label，以便於配置，部署等管理工作。例如：
- 部署不同版本的應用到不同的環境中
- 或者監控和分析應用（日志記錄，監控，報警等）。

通過多個Label的設置，我們就可以多維度的Pod或其他對象進行精細化管理。一些常用的Label示例如下：

```
relase: stable, canary
environment: dev, qa, production
tier: frontend, backend, middleware
```

## Annotation (k8s)
可以將Kubernetes資源對象關聯到任意的非標識性元數據。使用客戶端（如工具和庫）可以檢索到這些元數據。

Label和Annotation都可以將元數據關聯到Kubernetes資源對象。
- Label主要用於選擇對象，可以挑選出滿足特定條件的對象。
- annotation 不能用於標識及選擇對象。

annotation中的元數據可多可少，可以是結構化的或非結構化的，也可以包含label中不允許出現的字符。
annotation和label一樣都是key/value鍵值對映射結構

以下列出了一些可以記錄在 annotation 中的對象信息：

- 聲明配置層管理的字段。使用annotation關聯這類字段可以用於區分以下幾種配置來源：客戶端或服務器設置的默認值，自動生成的字段或自動生成的 auto-scaling 和 auto-sizing 系統配置的字段。
- 創建信息、版本信息或鏡像信息。例如時間戳、版本號、git分支、PR序號、鏡像哈希值以及倉庫地址。
- 記錄日志、監控、分析或審計存儲倉庫的指針
- 可以用於debug的客戶端（庫或工具）信息，例如名稱、版本和創建信息。
- 用戶信息，以及工具或系統來源信息、例如來自非Kubernetes生態的相關對象的URL信息。
- 輕量級部署工具元數據，例如配置或檢查點。
- 負責人的電話或聯系方式，或能找到相關信息的目錄條目信息，例如團隊網站。

annotation 對特定服務做配置
默認的 nginx 配置未必適合我們的服務，訪問 [Nginx Configuration](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/) 可以看到 ingress-nginx 所提供的三種 nginx 配置方式。
- 其中 ConfigMaps 可以實現對 nginx 默認配置的修改
- ingress annotation 則可以實現對特定 ingress 進行配置。
- Custom template: when more specific settings are required, like open_file_cache

所以這邊用 annotation 對 nginx 特別設定，是種既定的做法！！
- https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/


## `Pod` 和 `Replica Set`
這兩個是什麼關係？
個典型的用例如下：

- 使用Deployment來創建ReplicaSet。ReplicaSet在後台創建pod。檢查啟動狀態，看它是成功還是失敗。
- 然後，通過更新Deployment的PodTemplateSpec字段來聲明Pod的新狀態。這會創建一個新的ReplicaSet，Deployment會按照控制的速率將pod從舊的ReplicaSet移動到新的ReplicaSet中。
- 如果當前狀態不穩定，回滾到之前的Deployment revision。每次回滾都會更新Deployment的revision。
- 擴容Deployment以滿足更高的負載。
- 暫停Deployment來應用PodTemplateSpec的多個修覆，然後恢覆上線。
- 根據Deployment 的狀態判斷上線是否hang住了。
- 清除舊的不必要的ReplicaSet。

## Service 
Service 是 Kubernetes 內定義的抽象化物件(object)，官方網站介紹描述它的基本(原始)用途。

> A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them.
>Kubernetes Service 是個抽象化的概念，主要定義了邏輯上的一群 Pod 以及如何存取他們的**規則**。

這邊同時也帶出幾個問題:
- 誰會使用 Service
- 什麼是邏輯上的一群 Pod
- 什麼是存取規則

(為求講解方便此處繪出 Service 作為存取的進入點，實際上並沒有稱為 Service 的程式在運作。)  
？也就是說，Service 只是一個規則set？

### 誰會使用 Service 
- 外部使用者會透過 Service 存取內部 Pod 以 (路徑 1 -> 2)
- 同集群其他的 Pod 也有可能需要存取 (路徑 3 -> 2)。

值得注意的是兩條路徑的存取方式以及存取的 IP 位址有所不同

### 什麼是邏輯上的一群 Pod 
> 簡單一句話描述就是: 帶著相同標籤、做類似事情的一群 Pod

當 Service 指定存取某些特定的標籤時，Label Selector 會根據 Pod 身上自帶的標籤進行分組，並回傳 Service 所要求的 Pod 資訊。

上圖右邊共有三種分別為黃、綠、藍色的 Pod，正代表著集群內有三種不同的 Pod 群組，當今天收到使用者請求時，便會將請求送至對應的藍色群組內的其中一個 Pod 進行處理

### 什麼是存取規則 
就是如何存取該服務的規則，比方說 TCP/UDP、Port 等等相關規則。  
Service 作為中介層，避免使用者和 Pod 進行直接連線，除了讓我們服務維持一定彈性能夠選擇不同的 Pod 來處理請求外，某種程度上亦避免裸露無謂的 Port 而導致資安問題。另外，也體現出雲服務架構設計中一個非常重要的觀念:

> 對於使用者而言，僅須知道有人會處理他們的請求，而毋須知道實際上處理的人是誰。

### Kunernetes Service 定義檔主要涵括三個主要元素:
- 服務元資料 (Metadata)
- 被存取的應用之標籤 (Label)
- 存取該服務的方式

#### 服務元資料 
簡單來說就是服務的名稱，讓其他人瞭解該服務的用途
```yaml
metadata:
  name: service-example
```
#### 被存取的應用之標籤 
由於每個 Pod 本身會帶有一至多個標籤，如何將使用者請求送到正確的 Pod，仰賴管理者設定的標籤是否得當。
比方說，今天有 Nginx 以及 Apache 兩種網頁伺服器在運作，維運人員希望將流量導向至 Nginx，只要
1. 在 Pod 的 spec.selector 設定一組 app: nginx 的標籤
2. 接著在 Service 內定義:
```yaml
spec:
  selector:
    app: nginx
```
#### 存取該服務的方式 
服務要開放給外界使用時，需要定義該服務的 Port、Protocol。以常見網頁伺服器來打比方:
- Port: 80、443
- Protocol: TCP
對應到 Kubernetes Service spec.ports 的寫法的話就是:
```yaml
spec:
  ports:
    - name: http
      # the port for client to access
      port: 80
      # the port that web server listen on
      targetPort: 80
      # TCP/UDP are available, default is TCP
      protocol: TCP
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
```
而完整的 spec.ports 共包含以下幾個欄位:
- name: 讓維運人員瞭解該埠用途
- port: 對外部開放的埠號
- targetPort: 實際 Pod 所開放的埠號
- protocol (optional): 該服務使用的協定目前有 TCP/UDP 兩種，預設為 TCP
- nodePort (optional): 此設定只有在 spec.type 為 NodePort 或 LoadBalancer 才會存在


## why need `dumb-init`
`dumb-init` 是 A minimal init system for Linux containers。  
 is a simple process supervisor and init `system designed to run as PID 1` inside minimal container environments (such as Docker).  

Something like `dumb-init` or `tini` can be used if you have a process that spawns new processes and you `doesn't have good signal handlers` implemented to catch child signals and stop your child if your process should be stopped etc.

If your process doesn't spawn new processes (e.g., Node.js) then this may not be necessary.

I guess that MongoDB, PostgreSQL, ... which may run child processes have good signal handlers implemented. `Otherwise there would have been zombie processes and someone had filed an issue to fix this`.


Dumb-Init進程信號處理
【編者的話】隨著Docker及Kubernetes技術發展的越來越成熟穩定，越來越多的公司開始將Docker用於生產環境的部署，相比起物理機上直接部署，多了一層Docker容器的環境，這就帶來一個問題：`進程信號接收與處理`  

在Docker中捕獲不到進程的結束信號，這給`進程異常的處理`帶來了麻煩，用Supervisor等進程管理工具也能夠解決這一問題，不過太“重”了，在容器時代追求“輕”的我們是不能接受的  
工具-Dumb-Init。  
站在容器的角度，由其運行的第一個程序的`PID 1`，
- 這個進程的重要的使命：傳遞信號讓子進程退出和等待子進程退出。
- 對於第一點，如果pid為1的進程，無法向其子進程傳遞信號，可能導致容器發送SIGTERM信號之後，父進程等待子進程退出。此時，如果父進程不能將信號傳遞到子進程，則整個容器就將無法正常退出，除非向父進程發送SIGKILL信號，使其強行退出，這就會導致一些退出前的操作無法正常執行，例如關閉數據庫連接、關閉輸入輸出流等。

上邊提到：PID為1的進程的一個重要使命是等待子進程退出，如果

  1. 一個進程中A運行了一個子進程B，
  2. 這個子進程B又創建了一個子進程C
  3. 若子進程B非正常退出（通過SIGKILL信號，並不會傳遞SIGKILL信號給進程C），那麽子進程C就會由進程A接管

一般情況下，我們在進程A中並不會處理對進程C的托管操作（進程A不會傳遞SIGTERM和SIGKILL信號給進程C），結果就導致了進程B結束了，倒是並沒有回收其子進程C，子進程C就變成了僵屍進程。  

父進程非正常退出（收到SIGKILL信號）（用了Supervisor類工具，可以將SIGKILL信號傳遞給子進程），若`子進程`中想收到SIGTERM信號，就可以**通過dumb-init來模擬**。

## wget with -O flag

> wget -O filename.html www.stackoverflow.com
Output:
A file named as filename.html

## gettext
gettext是GNU國際化與在地化（i18n）函式庫。它常被用於編寫多語言程式。
網站	http://www.gnu.org/software/gettext/

## set
那麼如何觀察目前 shell 環境下的所有變數呢？很簡單啊，就用 `set` 即可！`set` 這個指令除了會將環境變數列出來之外，其他我們的自訂變數，與所有的變數，都會被列出來喔！資訊多好多。

## roffe/kubectl
看到這個 minimal docker images （下載 1M+），可能就是 copy past 這份吧，內容一模一樣。
- https://hub.docker.com/r/roffe/kubectl/dockerfile