+++
title = 'My First Post'
date = 2024-05-30T20:44:42+08:00
draft = true
+++



# 802.11 BSS Transition Management
*作者：Jakcson*
*更新日期：20240428*

BSS Transition Management 是 802.11v-2011 修正案中提出的，其主要功能是在需要网络负载均衡或 BSS Termination 期间可以持续提供服务，无需重新建立连接和中断网络服务，也就是提供漫游服务。


## 一、如何工作

<img src="https://img2024.cnblogs.com/blog/3437036/202404/3437036-20240427185158882-1508750860.png" width="800px" />

抛开上图中其他元素，我们只需要关注 frame 的交互过程。此过程只涉及三种帧：
* BTM Query(optional)
* BTM Request
* BTM Response(optional)

其中仅 request 是必须的。这也就提供了两种工作方式：
1. **Unsolicited request**：如果 AP 检测到 STA 的 RSSI 低于 RSSI threshold，它可以主动发送一个 BTM request 给 STA，而不需要等待 STA 发送 query。
2. **Solicited request**：如果 STA 检测到当前 AP 的 RSSI 过低，并且发现有更好 AP，可以发送一个 BTM query 给 AP，AP 分析之后发送一个 BTM request 给该 STA。


## 二、frame 细节


### （一） BTM QUERY
<img src="https://img2024.cnblogs.com/blog/3437036/202404/3437036-20240427212304944-1792272568.png" width="800px" />

* Dialog Token：该字段由 STA 生成的一个非零值，用来标记一次对话，因此该 Query 以及后续的 Request 和 Response 均为该值。
* BSS Transition Query Reason：STA 请求 BTM 的原因。
* BSS Transition Candidate List Entries：候选名单，该字段可以为空或者为数个 Neighbor Report elements。
<img src="https://img2024.cnblogs.com/blog/3437036/202404/3437036-20240427212742658-1684230867.png" width="800px" />
<img src="https://img2024.cnblogs.com/blog/3437036/202404/3437036-20240427212811740-1851167600.png" width="800px" />
****



### （二） BTM REQUEST
<img src="https://img2024.cnblogs.com/blog/3437036/202404/3437036-20240427212321137-2038913131.png" width="800px" />

* Dialog Token：若该 request 是回应 query，则该字段的值是 query 中 Dialog Token 的值；如果该 request 是由 AP 发起的，则该值是 AP 选择的一个非零值。
  * 若该 BTM 过程是由 STA 发起的，即 AP 在收到 BTM Query 之后发送的 BTM Request，则该值为 BTM Query 中所带的值。
  * 若该 BTM 过程是由 AP 发起的，该值为 AP 生成的一个非 0 值。

* Request Mode:该字段指明了 BTM 的一些工作方式，由以下字段决定：
<img src="https://img2024.cnblogs.com/blog/3437036/202404/3437036-20240427215234526-585313956.png" width="800px" />
  * Preferred Candidate List Included：是否包含有 Candidate List 即候选名单。
  * Abridged：该字段指明接收方如何处理未列在 Candidate List 中的 BSSIDs。
    * 置为 1：所有未出现在 Candidate List 中的 BSSIDs 的 preference value 为 0，即不要选择。
    * 置为 0：AP 对所有未出现在 Candidate List 中的 BSSIDs 不提供意见，由 STA 自行选择。
  * Disassociation Imminent：接收方是否立即断开与当前 AP 的连接。
  * BSS Termination Included：是否包含有 BSS Termination Duration 字段。
  * ESS Disassociation Imminent：是否包含由 Session Information URL 字段。
* The Disassociation Timer：表明多久后 AP 会发送一个 Disassociation 帧：
  * 0：表示 AP 也不知何时会发送解除关联。
  * Non-0：这里的单位不是时间单位，而是 AP 在解除关联之前会发送的信标次数，即 beacon transmission times（TBTTs）。因为实际时间为信标传送时间与该值的积。
* Validity Interval：Candidate List 候选名单的有效时间，单位同上。
* BSS Termination Duration：当 BSS Termination Included 为 1 时，该字段为一个 BSS Termination Duration subelement。
* Session Information URL：*后续补充*
* BSS Transition Candidate List Entries：候选者名单，该字段为空或数个 Neighbor Report elements。

在 ieee802.11 中，对于 BSS Transition Candidate List Entries 有这样一句描述：
> "If the STA has no information in response to the BSS Transition Management Query frame or in an unsolicited BSS Transition Management Request frame, the Neighbor Report elements are omitted and the Preferred Candidate List Included bit is 0."

然而在实际应用中，对于 unsolicited BTM Request 即使携带 Candidate List 也似乎也是可以工作的。

### （三） BTM RESPONSE
<img src="https://img2024.cnblogs.com/blog/3437036/202404/3437036-20240427212345788-2041224150.png" width="800px" />

* Dialog Token：同上。
* BTM Status Code：对于 BTM request，如果 STA 会漫游到其他 BSS ，status code 为 0；如果希望继续希望与当前 BSS 保持连接，则为下面任意一个 Reject Code。
<img src="https://img2024.cnblogs.com/blog/3437036/202404/3437036-20240427224204064-1271601185.png" width="800px" />
* BSS Termination Delay：Status Code 为 5 时，该字段为希望 AP 延迟 termination 的时间，单位分钟。
* Target BSSID：Status Code 为 0 时，该字段为漫游的目的 BSSID。
* BSS Transition Candidate List Entries：Status Code 置为 6 时，该字段不为空，为 Non-AP Sta 向 AP 提供的候选者名单。

# 三、其他细节
1. BTM 功能是可选功能，对于实现了 BTM 的 STA，应当将其 Extended Capabilities elements 中的 Transition 字段置为 1，这样 AP 可以知晓该 STA 具备 BTM 功能。

2. STA 发送 query 时理应包含 Candidate List。
3. 如果 STA 仅仅是发送一份 Candidate List，其 Query Reason 应置为 19，Preferred BSS Transition Candidate List Included.对于 Candidate List Neighbor Report element 中的 Preference 字段的取值范围为 1-255，0 是一个保留值。取值越高表示优先级越高。

4. AP 在回应 Query 时，Request 中理应包含 Candidate List。此处的 Neighbor Report element 中的 Preference 为 0 时，与 Query 有所不同，表示该 BSS 被排除。STA 应当避免关联到与排除的 BSS。Preference 字段只在 validity interval 字段有效期间有效。
5. 一旦 AP 收到的 Query 或 Response 带有 Candidate List。AP 应当评估 STA 发送的 Candidate List，并在之后发送的 Request 中 Candidate List 中应至少一个 Preference 非 0 的候选 BSS。

6. Response 中 Status Code 可返回的值，根据 request 的不同而有所不同：
* 当 BSS Termination Included 为 1 时，即 AP 即将终止：
	* 0
	* 4
	* 5

* 当 Disassociation Imminent 和 Preferred Candidate List Included 字段均为 0 时：
	* 0
	* 2
	* 3
	* 6
	* 7
	* 8

7. 当 AP 收到的 Response 中 Status Code 为 2 时，AP 应该发送一个 Disassociation Imminent 和 Preferred Candidate List Included 均为 0 的 request，让 STA 有充足的时间来执行其扫描过程。



# 参考
本文大多参考 802.11-2016 规范，对其中 BTM 相关内容进行解读，如有解读错误或不当还请大家在评论区指正。推荐大家阅读一下章节进一步理解：
* 4.3.18.3
* 6.3.59
* 9.6.14.8
* 9.6.14.9
* 9.6.14.10
* 11.24.7.4
