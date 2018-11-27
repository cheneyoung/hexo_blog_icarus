---
title: Unique-Email-Addresses
comments: true
author: zuoyang
type: 原创
toc: true
categories:
  - leetcode
tags:
  - C++
  - python
  - string
abbrlink: 486
date: 2018-11-28 00:09:41
---

# 问题描述：

Every email consists of a local name and a domain name, separated by the @ sign.

For example, in `alice@leetcode.com`, `alice` is the local name, and `leetcode.com` is the domain name.

Besides lowercase letters, these emails may contain `'.'`s or `'+'`s.

If you add periods (`'.'`) between some characters in the **local name** part of an email address, mail sent there will be forwarded to the same address without dots in the local name.  For example, `"alice.z@leetcode.com"` and `"alicez@leetcode.com"` forward to the same email address.  (Note that this rule does not apply for domain names.)

If you add a plus (`'+'`) in the **local name**, everything after the first plus sign will be **ignored**. This allows certain emails to be filtered, for example `m.y+name@email.com` will be forwarded to `my@email.com`.  (Again, this rule does not apply for domain names.)

It is possible to use both of these rules at the same time.

Given a list of `emails`, we send one email to each address in the list.  How many different addresses actually receive mails? 

 

**Example 1:**

```shell
Input: ["test.email+alex@leetcode.com","test.e.mail+bob.cathy@leetcode.com","testemail+david@lee.tcode.com"]
Output: 2
Explanation: "testemail@leetcode.com" and "testemail@lee.tcode.com" actually receive mails
```

 

**Note:**

- `1 <= emails[i].length <= 100`
- `1 <= emails.length <= 100`
- Each `emails[i]` contains exactly one `'@'` character.



# 解决思路：

根据题设的描述可以知道，对于给定的一个email，可以通过下面的两种规则对`localname`(也就是`@`符号前面的字符串)进行过滤，`domain`保持不变：

1. 所有的`.`都可以忽略
2. `+`后面的字符串全部忽略

基于这两个规则将一个字符串数组进行处理，然后统计处理之后的字符串数组中互不相同的email地址的个数。



# AC代码：

- C++

```C++
class Solution {
public:
    /* 按照上述规则处理email字符串 */
    string processEmail(string email) {
        int at = email.find_first_of('@',0);
        string domain = email.substr(at+1);
        string localname_s;
        if (at != string::npos) {
            localname_s = email.substr(0,at);
        }
        int plus = localname_s.find_first_of('+',0);
        string localname_plus;
        if (plus != string::npos) {
            localname_plus = localname_s.substr(0,plus);
        }
        string localname = "";
        for (int i = 0; i < localname_plus.length(); i++) {
            if (localname_plus[i] != '.') {
                localname += localname_plus[i];
            }
        }
        return localname+"@"+domain;
    }
    
    int numUniqueEmails(vector<string>& emails) {
        if (emails.size() == 0) {
            return 0;
        }
        vector<string> distinct_emails;
        int distinct_email_num = 0;
        bool flag = false;
        for(int i = 0; i < emails.size(); i++) {
            for (int j = 0; j < distinct_emails.size(); j++) {
                if (processEmail(emails[i]) == distinct_emails[j]) {
                    flag = true;
                    break;
                }
            }
            if (flag == false) {
                distinct_emails.push_back(processEmail(emails[i]));
                distinct_email_num ++;
            } else {
                flag = false;
            } 
        }
        return distinct_email_num;
    }
};
```



- python3

```python
class Solution:
    def processEmail(self,email):
        localname = email.split('@')[0]
        domain = email.split('@')[1]
        localname_plus = localname.split('+')[0]
        local = ""
        for part in localname_plus.split('.'):
            local = local + part
        return local+"@"+domain 
    
    def numUniqueEmails(self, emails):
        """
        :type emails: List[str]
        :rtype: int
        """
        distinct_email_num = 0
        distinct_emails = []
        if len(emails) == 0:
            return 0
        for email in emails:
            if self.processEmail(email) not in distinct_emails:
                distinct_emails.append(self.processEmail(email))
                distinct_email_num += 1
        return distinct_email_num
```

