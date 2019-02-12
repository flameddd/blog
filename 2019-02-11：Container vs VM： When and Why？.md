## Container vs VM： When and Why？
## 來源： https://igene.tw/container-vs-vm

Container 在這幾年來快速竄紅，憑的是其**輕量化和快速部署的彈性**。與其虛擬化整個硬體，Container 透過 **cgroups** 跟 **namespace** 將 **host OS 的資源做隔離**，也有人稱之為 OS 的虛擬化。 Container 共用 host 上的 kernel，只有其獨立的 binary 跟 libraries，所以比起 VM 在 image 大小跟啟動速度都快的許多。

### Container 的優點：
 - Image 很小：因為跟 host 共用 kernel，只需要提供需要的 OS 元件跟 libraries。
 - 速度：部署極快，通常花數秒就可以生成一個 container。
 - 可攜性：在正確的 kernel 下 ，container 可以容易在不同 host 或環境中移動。
 - CI/CD：container 因其可攜性跟速度在 CI/CD 的建置相對 VM 來得容易。
 - 週期管理：container 需要更新只需利用新的 image 重新啟動即可。

#### 需要解決的問題：
 - 安全性：跟 host 共用 kernel，安全性比起虛擬化整個硬體來得差。
 - 無法選擇 OS：同樣因為跟 host 共用 kernel 的關係，因此沒辦法在每個 container 都運行不同的 kernel。
 - 需要比較複雜的網路：container 通常切分成 micro-service 的方式做部署，在各元件中的網路連結會比較複雜。

VM 相對 container 是比較老跟成熟的技術。VM 會虛擬化整個硬體層並在上面安裝 OS，使用起來比較像一台獨立的系統。

### VM 的優點：
 - 整個系統的虛擬化：這讓 VM 在使用上跟 baremetal server 沒有什麼差別，能夠比較直覺的操作。
 - 不需要拆分應用程式：由於使用上跟 baremetal server 相同，可以不需要大幅更改應用程式的架構。
 - 安全性：由於整個硬體層都是虛擬化的，安全跟隔離性相對 container 好很多。
 - OS 選擇的彈性：在 VM 上可以自由選擇安裝不同的 OS

#### VM 的缺點：
 - 大小：VM 的 image 大小很大，通常是 GB 以上。
 - 啟動速度：VM 的啟動可能需要花個數分鐘，對需要快速增加應對需求的場合就沒有那麼好。
 - 運行速度：VM 由於需要虛擬化硬體層，效能相較於 baremetal server 會有一些減損。


## 使用時機
### Container 使用時機
 使用 Baremetal Container 前應該先評估過以下條件：
 - Application 已經 microservice 化：Container 在使用的 best practice 需要使用 microservice 的架構。
 - 運行的是 trusted code：如果運行的是 trusted code 可以避免各種 kernel 漏洞被利用。
 - 有完善的 baremetal 部署機制：Container 還是需要跑在裝有 OS 的 baremetal server 上，這時就需要一個完整的 baremetal provisioning 機制。
 - 沒有特殊的網路需求：現在 Container Orchestration Engine 如 Kubernetes 對於特殊網路需求例如多網卡等受限於 CNI spec 支援並不是很好。

### VM 使用時機
  VM 的部分則有相當不一樣的條件需要評估：

 - Application 不需要快速 scale out：VM 在生成的速度比 container 慢許多，如果需要快速的 scale out 因應 load spike，效果沒有 container 好。
 - 運行 untrusted code：運行 untrusted code 會建議在 VM 上，有比較完善的隔離。這也是為什麼 Public Cloud 上跑的還是隔了一層 VM，因為服務提供者無法確認使用者會在上面運行什麼程式。
 - 需要 hard multi-tenancy：hard multi-tenancy 在 Container Orchestration Engine 上還沒有被實現。
 - 有特殊的網路需求：VM 在動態 plug-in, plug-out 新網卡、多網卡等應用上發展比較完善。

目前 VM 最大宗的使用需求還是在
 - 傳統應用程式：因為並非 microservice 架構，使用 container 意義不大
 - Public Cloud：使用者的隔離性還有系統的安全性考量後的選擇。


## 結論
現今大多數情況下
 - VM 仍然作為 Infrastructure 的基礎
 - Container 則作為 Application 的基礎。
 Container 跟 VM 都有其適合應用的時機，兩者目前還是屬於可以共存的並且互相應用的狀態。


### others
#### 虛擬化技術簡介
  在 x86 世界，虛擬化技術可簡單分為 **Hosted** 及 **Native** 這兩類。

- Hosted：是在現行OS (e.g. Windows, Linux)上運行，事實上它就是一支AP；VMWare Workstation，VMWare Player 及免費的VMWare server (以前稱為GSX server)都屬於此類。

- Native：直接在硬體上運行(又稱Bare-metal)
  - 底下不需要另一個OS；
  - 它的效能比 Hosted 好，也就是在同樣硬體資源下，Native 可支援更多個虛擬機(VM)。
  
VMWare ESX server 屬於此類。有些Native 的虛擬化技術，在其上運行的Guest OS必須經過修改才能使用，這類虛擬化技術稱為Paravirtualization，市面上Xen 此產品即屬於此類；

