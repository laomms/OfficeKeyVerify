# OfficeKeyVerify
检测Office绑定密钥是否有效

#httpwebrequest实现

#支持自动登录账号

#支持批量检测,批量绑定

#支持导出已兑换密钥

#自带数据库存储检测结果

#支持下载office软件

![image](https://github.com/laomms/OfficeKeyVerify/blob/master/1pc.gif)

放上登录过程源码:    

```c
          //第一步：获取登录按钮的跳转地址
            var url1 = "https://setup.office.com";
            head1.Add("Accept-Encoding:gzip, deflate");
            //mycookiecontainer用于存储每个步骤返回的ResponseHeader中的SetCookie也用于下一步Request的CookieContainer,redirect_geturl用于存储头文件中的"Location"跳转地址
            var ResponseString1 = RequestGet(url1, headaccept, contentype, url1, head1, mycookiecontainer, out redirect_geturl);
            string url2 = "";
            HtmlAgilityPack.HtmlDocument htmldocument1 = new HtmlAgilityPack.HtmlDocument();
            htmldocument1.LoadHtml(ResponseString1);
            foreach (HtmlNode divNode in htmldocument1.DocumentNode.SelectNodes("//div[@class='col-xs-12 margin-top-16 nopaddding']"))//登录框
            {
                HtmlNode[] nodes = divNode.SelectNodes(".//a").ToArray();
                foreach (HtmlNode item in nodes)
                {
                    if (item.Id== "btnSignin")//登录按ID
                    {
                        url2 = HttpUtility.HtmlDecode(HttpUtility.UrlDecode(item.GetAttributeValue("href", ""))); //获取下一步的跳转链接
                        break;
                    }
                }              

            }
            foreach (HtmlNode input in htmldocument1.DocumentNode.SelectNodes("//input"))  //获取CorrelationId
            {
                if (input.Id == "hdnRequestCorrelationId")
                {
                    CorrelationId = input.GetAttributeValue("value", "");
                    break;
                }
               
            }

            ///////////////////////////////////////////////////////////////////////
            //第二步：获取登录参数
            //CookieContainer cookies = new CookieContainer();
            var ResponseString2 = RequestGet(url2, headaccept, contentype, url1, head1, mycookiecontainer, out redirect_geturl);
            string originalRequest = string.Empty, canary = string.Empty, urlPost = string.Empty, correlationId = string.Empty, sessionId = string.Empty, nonce = string.Empty, flowToken = string.Empty, sCtx = string.Empty, country = string.Empty, apiCanary = string.Empty, hpgact = string.Empty, hpgid = string.Empty;
            var url3 = "";
            var url4 = "";
            var uaid = "";
            //在返回的网页中正则出json格式,获取各个参数及跳转链接
            MatchCollection match2 = Regex.Matches(HttpUtility.HtmlDecode(HttpUtility.UrlDecode(ResponseString2)), @"\{(?:[^\{\}]|(?<o>\{)|(?<-o>\}))+(?(o)(?!))\}", RegexOptions.Multiline | RegexOptions.IgnoreCase);
            foreach (Match match in match2)
            {
                try
                {
                    JToken token = JObject.Parse(match.Value);
                    if (token.SelectToken("$.Bq") != null)
                    {
                        url3 = (string)token.SelectToken("$.Bq");   //获取下一步的跳转链接
                    }
                    if (token.SelectToken("$.urlPost") != null)
                    {
                        url4 = (string)token.SelectToken("$.urlPost");
                    }
                    if (token.SelectToken("$.correlationId") != null)
                    {
                        uaid = (string)token.SelectToken("$.correlationId");
                    }
                    if (token.SelectToken("$.hpgid") != null)
                    {
                        hpgid = (string)token.SelectToken("$.hpgid");
                    }
                    if (token.SelectToken("$.hpgact") != null)
                    {
                        hpgact = (string)token.SelectToken("$.hpgact");
                    }
                    if (token.SelectToken("$.sFTTag") != null)
                    {
                        var sFTTag = (string)token.SelectToken("$.sFTTag");
                        HtmlAgilityPack.HtmlDocument htmldocument2 = new HtmlAgilityPack.HtmlDocument();
                        htmldocument2.LoadHtml(sFTTag);
                        foreach (HtmlNode input in htmldocument2.DocumentNode.SelectNodes("//input"))
                        {
                            flowToken = input.GetAttributeValue("value","");
                        }
                    }
                }
                catch
                {
                }
            }
           //取cookie集中的ip地址备用其他步骤验证
            var table = (Hashtable)mycookiecontainer.GetType().InvokeMember("m_domainTable", BindingFlags.NonPublic | BindingFlags.GetField | BindingFlags.Instance,null, mycookiecontainer, null);
            foreach (var key in table.Keys)
            {
                var item = table[key];
                var items = (ICollection)item.GetType().GetProperty("Values").GetGetMethod().Invoke(item, null);
                foreach (CookieCollection cc in items)
                {
                    foreach (Cookie cookie in cc)
                    {
                        if (cookie.Name== "MSCC")
                        {
                            ipaddr = cookie.Value.Substring(0, cookie.Value.IndexOf("-"));
                            break;
                        }
                    }
                }

            }
            //登录的各个必要参数
            var ANON = "";
            var pprid = "";
            var wbids = "";
            var NAP = "";
            var t = "";
            var wbid = "";
            
            var url5 = "";           
            string txtBox1 = "";
            textBox1.Invoke(new MethodInvoker(delegate { txtBox1 = textBox1.Text; }));  //微软登录邮箱
            string txtBox2 = "";
            textBox2.Invoke(new MethodInvoker(delegate { txtBox2 = textBox2.Text; }));  //登录密码
            if (url3 != "")
            {
                ///////////////////////////////////////////////////////
                ///第三步：输入登录邮箱 
                WebHeaderCollection head3 = new WebHeaderCollection()
                           {
                               {"Accept-Encoding:gzip, deflate"},
                               {"client-request-id", uaid},
                               {"hpgid",hpgid},
                               {"hpgact","0"}
                           };
               
                //post的josn格式
                var postdata3 = "{\"username\":\"" + txtBox1 + "\",\"uaid\":\"" + uaid + "\",\"isOtherIdpSupported\":false,\"checkPhones\":false,\"isRemoteNGCSupported\":true,\"isCookieBannerShown\":false,\"isFidoSupported\":false,\"forceotclogin\":false,\"otclogindisallowed\":true,\"isExternalFederationDisallowed\":false,\"flowToken\":\"" + flowToken + "\"}";
                var ResponseString3 = RequestPost(url3, "application/json", "application/json; charset=UTF-8", url2, head3, postdata3, mycookiecontainer, out redirect_posturl);

            }
            //登录
            var postdata4 = "i13=0&login=" + HttpUtility.UrlEncode(txtBox1) + "&loginfmt=" + HttpUtility.UrlEncode(txtBox1) + "&type=11&LoginOptions=3&lrt=&lrtPartition=&hisRegion=&hisScaleUnit=&passwd=" + HttpUtility.UrlEncode(txtBox2) + "&ps=2&psRNGCDefaultType=&psRNGCEntropy=&psRNGCSLK=&canary=&ctx=&hpgrequestid=&PPFT=" + flowToken + "&PPSX=Passpo&NewUser=1&FoundMSAs=&fspost=1&i21=0&CookieDisclosure=0&IsFidoSupported=0&i2=6&i17=0&i18=&i19=78649";
            var ResponseString4 = RequestPost(url4, headaccept, contentype, url3, head1, postdata4, mycookiecontainer, out redirect_posturl);

            HtmlAgilityPack.HtmlDocument htmldocument4 = new HtmlAgilityPack.HtmlDocument();
            htmldocument4.LoadHtml(ResponseString4);
            if (ResponseString4.Contains("<form name=\""))  //是否正确返回登录成功参数
            {
                foreach (HtmlNode formNode in htmldocument4.DocumentNode.SelectNodes("//form"))
                {
                    if (formNode.Name == "form")
                    {
                        url5 = formNode.GetAttributeValue("action", "");     //获取下一步的跳转链接
                        break;
                    }
                }
                //获取登录成功的各个必要参数值
                foreach (HtmlNode input in htmldocument4.DocumentNode.SelectNodes("//input"))
                {
                    if (input.GetAttributeValue("name", "") == "wbids")
                    {
                        wbids = input.GetAttributeValue("value", "");
                    }
                    if (input.GetAttributeValue("name", "") == "wbid")
                    {
                        wbid = input.GetAttributeValue("value", "");
                    }
                    if (input.GetAttributeValue("name", "") == "pprid")
                    {
                        pprid = input.GetAttributeValue("value", "");
                    }
                    if (input.GetAttributeValue("name", "") == "NAP")
                    {
                        NAP = input.GetAttributeValue("value", "");
                    }
                    if (input.GetAttributeValue("name", "") == "t")
                    {
                        t = input.GetAttributeValue("value", "");
                    }
                }
            }
            else
            {
                this.Invoke(new UpdateMyDelegatedelegate(UpdateMessage), "账号或密码错误!");
                return;
            }            
            //登录成功  mycookiecontainer可以用于其他步骤
            var postdata5 = "wbids=" + wbids + "&pprid=" + pprid + "&wbid=" + wbid + "&NAP=" + NAP + "&=" + ANON + "&t=" + t;
            var ResponseString5 = RequestPost(url5, headaccept, contentype, url4, head1, postdata5, mycookiecontainer, out redirect_posturl);
            Regex metaTag = new Regex(@"<meta[\s]+[^>]*?name[\s]?=[\s""']+(.*?)[\s""']+content[\s]?=[\s""']+(.*?)[""']+.*?>");
            Dictionary<string, string> metaInformation = new Dictionary<string, string>();
            foreach (Match m in metaTag.Matches(ResponseString5))
            {
                metaInformation.Add(m.Groups[1].Value, m.Groups[2].Value);
                if (m.Groups[1].Value == "ms.ctid")
                {
                    CorrelationId = m.Groups[2].Value;
                    break;
                }
            }
            this.Invoke(new UpdateMyDelegatedelegate(UpdateMessage),"登录成功!");
```
