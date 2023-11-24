> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [susam.net](https://susam.net/blog/timing-with-curl.html)

> By Susam Pal on 10 Jul 2010

By **Susam Pal** on 10 Jul 2010

Here is a command I use often while measuring why an HTTP request is taking too long:

```
curl -L -w "time_namelookup: %{time_namelookup}
time_connect: %{time_connect}
time_appconnect: %{time_appconnect}
time_pretransfer: %{time_pretransfer}
time_redirect: %{time_redirect}
time_starttransfer: %{time_starttransfer}
time_total: %{time_total}
" https://example.com/
```

Here is the same command written as a one-liner, so that I can copy it easily from this page with a triple-click whenever I need it in future:

```
curl -L -w "time_namelookup: %{time_namelookup}\ntime_connect: %{time_connect}\ntime_appconnect: %{time_appconnect}\ntime_pretransfer: %{time_pretransfer}\ntime_redirect: %{time_redirect}\ntime_starttransfer: %{time_starttransfer}\ntime_total: %{time_total}\n" https://example.com/
```

Here is how the output of the above command typically looks:

```
$ curl -L -w "namelookup: %{time_namelookup}\nconnect: %{time_connect}\nappconnect: %{time_appconnect}\npretransfer: %{time_pretransfer}\nstarttransfer: %{time_starttransfer}\ntotal: %{time_total}\n" https://example.com/
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
...
</html>
time_namelookup: 0.001403
time_connect: 0.245464
time_appconnect: 0.757656
time_pretransfer: 0.757823
time_redirect: 0.000000
time_starttransfer: 0.982111
time_total: 0.982326


```

In the output above, I have omitted most of the HTML output and replaced the omitted part with ellipsis for the sake of brevity.

The list below provides a description of each number in the output above. This information is picked straight from the manual page of curl 7.20.0. Here are the details:

*   _time_namelookup:_ The time, in seconds, it took from the start until the name resolving was completed.
    
*   _time_connect:_ The time, in seconds, it took from the start until the TCP connect to the remote host (or proxy) was completed.
    
*   _time_appconnect:_ The time, in seconds, it took from the start until the SSL/SSH/etc connect/handshake to the remote host was completed. (Added in 7.19.0)
    
*   _time_pretransfer:_ The time, in seconds, it took from the start until the file transfer was just about to begin. This includes all pre-transfer commands and negotiations that are specific to the particular protocol(s) involved.
    
*   _time_redirect:_ The time, in seconds, it took for all redirection steps include name lookup, connect, pretransfer and transfer before the final transaction was started. time_redirect shows the complete execution time for multiple redirections. (Added in 7.12.3)
    
*   _time_starttransfer:_ The time, in seconds, it took from the start until the first byte was just about to be transferred. This includes time_pretransfer and also the time the server needed to calculate the result.
    
*   _time_total:_ The total time, in seconds, that the full operation lasted. The time will be displayed with millisecond resolution.
    

An important thing worth noting here is that the difference in the numbers for `time_appconnect` and `time_connect` time tells us how much time is spent in SSL/TLS handshake. For a cleartext connection without SSL/TLS, `time_appconnect` is reported as zero. Here is an example output that demonstrates this:

```
$ curl -L -w "time_namelookup: %{time_namelookup}\ntime_connect: %{time_connect}\ntime_appconnect: %{time_appconnect}\ntime_pretransfer: %{time_pretransfer}\ntime_redirect: %{time_redirect}\ntime_starttransfer: %{time_starttransfer}\ntime_total: %{time_total}\n" http://example.com/
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
...
</html>
time_namelookup: 0.001507
time_connect: 0.247032
time_appconnect: 0.000000
time_pretransfer: 0.247122
time_redirect: 0.000000
time_starttransfer: 0.512645
time_total: 0.512853


```

Also note that `time_redirect` is zero in both outputs above. That is because no redirection occurs while visiting [example.com](https://example.com/). Here is another example that shows how the output looks when a redirection occurs:

```
$ curl -L -w "time_namelookup: %{time_namelookup}\ntime_connect: %{time_connect}\ntime_appconnect: %{time_appconnect}\ntime_pretransfer: %{time_pretransfer}\ntime_redirect: %{time_redirect}\ntime_starttransfer: %{time_starttransfer}\ntime_total: %{time_total}\n" https://susam.net/blog
<!DOCTYPE HTML>
<html>
...
</html>
time_namelookup: 0.001886
time_connect: 0.152445
time_appconnect: 0.465326
time_pretransfer: 0.465413
time_redirect: 0.614289
time_starttransfer: 0.763997
time_total: 0.765413


```

When faced with a potential latency issue in web services, this is often one of the first commands I run several times from multiple clients because the results from this command help to get a quick sense of the layer that might be responsible for the latency issue.