## AWS costs every programmer should know
### Jun 9, 2019 • David Hatanian
#### https://david-codes.hatanian.com/2019/06/09/aws-costs-every-programmer-should-now.html


專案到某個規模，會有很多事情要考量
- 如何快速獲得市場
- 我的設計能否支撐我預期的 scale

在 scale 的 issue 中，其中就是要想你的 infrastructure 的 cost  
這邊作者提出他常用的一些參考數字  
並非精準的數字，但可以用來評估了  
有些公司還有 AWS 的 discounts，所以數字也有機會蠻大的差異  


### Compute 運算能力

整理表 ec2instances.info
- https://www.ec2instances.info/

這邊的參考數字是 `eu-west-1 region`  

|  | Median monthly cost |
| :------ | ------: |
|1 modern vCPU (4 AWS ECUs)|58 $/month|
|With 1 year convertible reservation (all up front)	|43 $/month|
|With 3 years convertible reservation (all up front)	|30 $/month|
|With spot pricing (estimated)	|30 $/month|

AWS represents the computing power of its machines in Elastic Compute Units, and 4 ECUs represent more or less the power of a modern CPU. So the prices above are for one CPU or core, not one instance.

Here’s the price of 1 ECU in $ per hour across all instance types I looked at:

### Storage

如果規劃是
- low latency
- high throughput
- planning to store everything in Redis

那除了上面 CPU 的 $ 之外，還要付 RAM 的費用

`Elasticache`
- is more or less twice as expensive for on-demand
- but prices drop quite quickly when looking at reserved instances.

|  | Median monthly cost |
| :------ | ------: |
|1 GB RAM|10 $/month|
|1 GB RAM 1 year convertible reservation (all up front)|8 $/month|
|1 GB RAM 3 years convertible reservation (all up front)|5 $/month|
|SSD|0.11 $/month|
|Hard Disk|0.05 $/month|
|S3|0.02 $/month|
|S3 Glacier|0.004 $/month|

這邊是純粹的 storage cost
- 所以要思考**你的 data usage patterns 是什麼？**
- How much CPU will you need to run that in-memory database 24/7?

S3 也是一樣
- how much will you pay in writing/reading requests?

### Bandwidth

要提供 data 給 end users 或需要跨區的，就要思考這邊

|Type of data transfer|Cost of transferring 1GB|
| :------ | ------: |
|EU/US region to any other region|0.02 $/GB|
|APAC region to any other region|0.09 $/GB|
|EU/US region to Internet|0.05 $/GB|
|APAC region to Internet|0.08 $/GB|
|Between two AZs in the same region|0.01 $/GB|
|Inside the same AZ|Free|


### AWS
- AWS 重要的不是**價格實惠**
- AWS 重要的是**架構**、**backup**、**emergency deployment**

### hacker news 一些討論
#### https://news.ycombinator.com/item?id=20138409

特定情境下（網站提供漫畫看）
- pushing over 1PB/month of images
- at the lowest rate of $0.02/GB with cloudfront/S3
  - it would be $20,000 per month
- bandwidth cost is stupid expensive on AWS

But they ended up paying nothing by using `cloudflare` who gave them unmetered bandwidth.  
until they got throttled earlier this year but they got out of that by upgrading to a measly $200/month plan  

這 case 跟很多 case 一樣，都說明的 CDN 大幅降低成本，聽到的幾乎都說可以省 70% 以上...  

關於 bandwidth
- Over 10tb of transfer you can (and should) negotiate some cheaper pre-committed pricing on bandwidth.

慢慢接觸吧，這些要碰的人才有感受