# Security Report

This page is the detail explanation of security breach for our website on 27/Nov/2018

## Method:

* the hack started by a big amount of request on xmlrpc.php to brute force the WordPress password. on 27 Nov 2018 the hackers gained access to the server.
* hacker uploaded a fake WooCommerce plugin to the WordPress. by doing that he become able upload other files
* hacker uploaded wp-loud.php to get the dB username and password.
* hacker uploaded wp-main.php to patch all the published post on the WordPress.
* hacker uploaded wp-register{uuid}.php which is an admin script. he used the admin script to patch the index.php.
* the script on the index.php on first run by the hacker start to call `http://opensite.ltd/api.php?g={xxx}` the `xxx` is a hardcoded number related to the hacked website.
    > for our case it was 9001. 
`http://opensite.ltd/api.php?g=9001`

* the API return a pipe (|) delimited text which consist of two base64 string.
    >the result for previous call is `LmJlc3R5c2NvbXB1dGVybWFsbC50b3A=|OTAwMC5iZXN0eXNjb21wdXRlcm1hbGwudG9w` which after decoding becomes `.bestyscomputermall.top|9000.bestyscomputermall.top`

* decoded version of each of the base64 code is a Url of another website. from here I will refer to them as Url[0] and Url[1].
* the Url[0] contains only the hostname and a sub domain. the script generate a url by concatenating 'www' with the xxx that we called in previous api call.
    >in our example url[0] become `www9001.bestyscomputermall.top`

* the Url[1] already contain a subdomain. the script will add 'www' to the start of that and it become a Url to get data information.
* the result will be a Url that will be called to get information about the files that need to be patched by calling the `http://url[0]/mapfile.txt` 
    >in our example `http://www9001.bestyscomputermall.top/mapfile.txt`
    >> the result of this call is single line string which contains `robots.txt` 

* the script then will call another endpoint to get information about patching. i will call this Url as data Url from now on.

    `http://{url[0]}/data1028.php?d=&g={xxx}&t={yyyy-mm-dd hh:ss}&u={file to patch}&h={website schema}&p=0.0&r={referer}&a={browser-agent}&l={website-language};q=0.9&i={current folder}&j=0&o={execution directory}|{path to root directory}`
    
    >in our example it's `http://www9001.bestyscomputermall.top/data1028.php?d=&g=9001&t=2018-11-30+02%3A41%3A24&u=%2Frobots.txt&h=http&p=0.0&r=&a=&l=en-US,en;q=0.9&i=/&j=0&o=/run_dir/|/home/testbargain/beta`
    >> this will result to the content that need to be written into the robots.txt

* the script on each call to the website check if it's search engine crawler or not. if it's, then it will call the data Url and pass the browser agent. in this case the data Url return a full webpage which replace the original content of the webpage.
    >in our example is `http://www9001.bestyscomputermall.top/data1028.php?d=&g=9001&t=2018-11-30+02:41:24&u=/index.php&h=http&p=0.0&r=&a=Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)&l=en-US,en;q=0.9&i=/&j=0&o=/run_dir/|/home/testbargain/beta` 
    >>the result of last call return some japanese website (as later it turned out it's a chinese company/person that is promoting japanese content atm)

    >>you can try the url in your browser

* the return result contains many `<a>` tag with href with empty host and to a imaginary Url like "http:////wagon.html". on next call when the crawler try to call the next page which in our example is `wagon.html` it will hijack the call and return another page related to the `wagon.html`.

    >in our example the call is `http://www9001.bestyscomputermall.top/data1028.php?d=&g=9001&t=2018-11-30+02:41:24&u=/wago.html&h=http&p=0.0&r=&a=Mozilla/5.0 (compatible; Googlebot/2.1; +http://www.google.com/bot.html)&l=en-US,en;q=0.9&i=/&j=0&o=/run_dir/|/home/testbargain/beta`
    >>you can try this url in your browser

* the index script on each call also check if the request is coming from yahoo.co.jp, google.jp and Bing. if the condition is true, the script will call the jump url and ask for some content. the content will be added to the actual website content in our example the Url is.

* a small script has written to extract all the possible Url which belong to the hacker from the `opensite.ltd`. 

   to be able to reproduce the calls to get the same Url from the website the xxx also is included. you should be able to call the `http://opensite.ltd/api.php?g={xxx}` to get the base64 text.

   the list of the URLs and whois are as follow.

|url[0] section |domain registrar abuse report email| owner name | email| xxx number|
|---------------|--------------|--------------|------------|---|
|`www9001.p-treff.info` | abuse@godaddy.com| - | - | 88, 99|
|`www9001.fashionwebs.wang`|DomainAbuse@service.aliyun.com | wang xiao ling | yakamoyoto@yahoo.co.jp|301,302|
|`ww9001.worldshop.ltd`| abuse@list.alibaba-inc.com | wang xiao ling | - | 303|
|`www9001.bestyscomputermall.top`|DomainAbuse@service.aliyun.com| wang xiao ling | yakamoyoto@yahoo.co.jp |9001|
|`www9001.goodstart.red`|Alibaba Cloud Computing Ltd. d/b/a HiChina| 王晓玲/王晓玲 | - | 9002,9003|
|`www9001.goodbeautifulworld.online`|domainabuse@service.aliyun.com| wang xiao ling | - | 9004|
|`www9001.beststart.fun` |DomainAbuse@service.aliyun.com|wang xiao ling | - |9005| 
|`www9001.fieryburning.vip` |DomainAbuse@service.aliyun.com| - | - | 9006|
|`www9001.practicalbicycleshop.mobi` |abuse@list.alibaba-inc.com| 王晓玲 | - | 9007|
|`www9001.populargoodsshop.ink` |DomainAbuse@service.aliyun.com| wang xiao ling | - | 9008|
|`www9001.bostondelivery.kim` |abuse@list.alibaba-inc.com|王晓玲 | - | 9009|
|`www9001.singaporeversion.tech` |domainabuse@service.aliyun.com| wang xiao ling | - | 9010|
|`www9001.floridaat.fun` |domainabuse@service.aliyun.com| wang xiao ling | - | 9011|
|`www9001.electronicsinternet.shop` |DomainAbuse@service.aliyun.com| - | - | 9012|
  
|url[1] section |registerd with| owner name | email| xxx number|
|---------------|--------------|--------------|------------|---|
|`www100.p-treff.info`||||88,89|
| `www300.fashionwebs.wang` ||||301,302,303|
|`www9000.bestyscomputermall.top`||||9001,9002,9003,9004,9005,9006,9007,9008,9009,9010,9011,9012|


the hacker with confident on the obfuscated code and complicated mechanism of their code trusted to expose their identity through the URLs. since there wasn't any automatic decompiler for the used obfuscation method hence a great manual effort has done to reverse engineer the algorithm.

detail information and logs and codes are available upon request.



