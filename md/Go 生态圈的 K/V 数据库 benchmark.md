> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [colobu.com](https://colobu.com/2019/03/05/go-kv-databases-benchmark/)

> Go 生态圈有好几个 K/V 数据库，我们经常用它来做我们的存储引擎，但是这些数据库引擎的性能如何呢？本文试图用性能而不是功能的数据考察这些数据库，我测试了几种场景： 并发写、并发读、单一写并发读、并发删除，得出了一些有趣的数据。

Go 生态圈有好几个 K/V 数据库，我们经常用它来做我们的存储引擎，但是这些数据库引擎的性能如何呢？本文试图用性能而不是功能的数据考察这些数据库，我测试了几种场景： 并发写、并发读、单一写并发读、并发删除，得出了一些有趣的数据。

测试在两台机器上测试的，一台机械硬盘，一台固态硬盘，使用 256 字节作为 value 值的大小，9 个字节作为 key 的大小，测试简单的读写删除操作，并没有测试批量读写操作。 每个测试 case 测试 1 分钟。

代码: [kvbench](https://github.com/smallnest/kvbench)

K/V 数据库
-------

*   [Rocksdb](https://github.com/facebook/rocksdb): RocksDB 是 Facebook 维护的高性能的嵌入式 K/V 数据库。它是 [LevelDB](https://github.com/google/leveldb) 的克隆版，针对多核、SSD 做了很多优化。 LSM tree 数据结构。
*   [badger](https://github.com/dgraph-io/badger): 一个纯 Go 实现的快速的嵌入式 K/V 数据库，针对 LSM tree 做了优化，在某些情况下可以取得比较好的性能。
*   [goleveldb](https://github.com/syndtr/goleveldb): leveldb 的纯 Go 实现，非 Google 的 C++ 版本
*   [bolt](https://github.com/boltdb/bolt): 一个广泛使用的 K/V 数据库，基本上不会有新的功能加入了。
*   [bbolt](https://github.com/etcd-io/bbolt): coreos 的 bolt 克隆版，继续维护和扩展 bolt 的功能。
*   [buntdb](https://github.com/tidwall/buntdb): 一个基于内存的 K/V 数据库，也可以落盘。
*   [cznic/kv](https://github.com/cznic/kv)： 基本上不维护了。
*   [pebble](https://github.com/petermattis/pebble): 一个性能优异的 K/V 数据库。
*   map (in-memory) with [AOF persistence](https://redis.io/topics/persistence): 基于 map 数据结构的数据库。
*   btree (in-memory) with [AOF persistence](https://redis.io/topics/persistence): 基于 btree 数据结构的数据库。

`btree`使用 btree 的数据结构，如果文件路径为`:memory:`, 则直接写内存，而不会存于硬盘文件中。`fsync`为 true 是会同步硬盘文件。  
`map`使用 map 的数据结构，如果文件路径为`:memory:`, 则直接写内存，而不会存于硬盘文件中。`fsync`为 true 是会同步硬盘文件。

当然，除了基本的 K/V 操作外，有些数据库还加入了事务、bucket(Column Families) 等额外的功能，这也是选型的时候要考虑的一个重要方面。

查看硬件环境
------

查看磁盘情况：`fdisk -l`  
查看磁盘型号：`smartctl -a /dev/sda`

下面两个命令不太准， 都显示 1:  
`cat /sys/block/sda/queue/rotational`  
`lsblk -d -o name,rota`

使用`MegaCli`工具:  
`/opt/MegaRAID/MegaCli/MegaCli64 -ldpdinfo -aall |grep 'Media Type'`

机械硬盘
----

硬盘是希捷的机械硬盘：

```
Vendor:               SEAGATE

Product:              ST900MM0168

User Capacity:        900,185,481,216 bytes [900 GB]

Rotation Rate:        10500 rpm
```

2 客 CPU, 20 个物理 CPU core， 开了超线程 40 个逻辑 CPU core。  
32G 内存。

### 不写盘 (nofsync)

**throughputs**

<table><thead><tr><th></th><th>badger</th><th>bbolt</th><th>bolt</th><th>leveldb</th><th>kv</th><th>buntdb</th><th>rocksdb</th><th>btree</th><th>btree/memory</th><th>map</th><th>map/memory</th></tr></thead><tbody><tr><td>get</td><td>474636</td><td>428163</td><td>467606</td><td>338100</td><td>17291</td><td>3051031</td><td>2433409</td><td>4966106</td><td>5485222</td><td>6740588</td><td>6555700</td></tr><tr><td>setmixed</td><td>1619</td><td>9469</td><td>7842</td><td>17220</td><td>309</td><td>6001</td><td>107852</td><td>17673</td><td>18513</td><td>18249</td><td>20234</td></tr><tr><td>getmixed</td><td>333540</td><td>127328</td><td>241168</td><td>344907</td><td>12359</td><td>359620</td><td>1976448</td><td>1275432</td><td>1519957</td><td>2240288</td><td>2597800</td></tr><tr><td>del</td><td>113590</td><td>4054</td><td>3523</td><td>219681</td><td>6379</td><td>188057</td><td>670461</td><td>461692</td><td>908052</td><td>797977</td><td>1405268</td></tr><tr><td>set</td><td>97441</td><td>11922</td><td>12414</td><td>130572</td><td>2267</td><td>99370</td><td>483053</td><td>203632</td><td>555432</td><td>243506</td><td>1120192</td></tr></tbody></table>

![](https://raw.githubusercontent.com/lslz627/PicGo/master/hdd-nofsync-throughputs.png)

`btree`和`map`相关的实现最好，因为都是直接的内存操作，而且操作也很简单。而且`map`更简单，通过 hash 直接找到对应对象。

对于其它的 K/V 数据库，相对来说 rocksdb 性能更好一些，而 buntdb 在只读的情况下性能不错。

**time (latency)**

<table><thead><tr><th></th><th>badger</th><th>bbolt</th><th>bolt</th><th>leveldb</th><th>kv</th><th>buntdb</th><th>rocksdb</th><th>btree</th><th>btree/memory</th><th>map</th><th>map/memory</th></tr></thead><tbody><tr><td>setmixed</td><td>617409</td><td>105605</td><td>127517</td><td>58071</td><td>3235598</td><td>166629</td><td>9271</td><td>56581</td><td>54015</td><td>54796</td><td>49421</td></tr><tr><td>del</td><td>220</td><td>6165</td><td>7096</td><td>113</td><td>3918</td><td>132</td><td>37</td><td>54</td><td>27</td><td>31</td><td>17</td></tr><tr><td>set</td><td>256</td><td>2096</td><td>2013</td><td>191</td><td>11023</td><td>251</td><td>51</td><td>122</td><td>45</td><td>102</td><td>22</td></tr><tr><td>get</td><td>52</td><td>58</td><td>53</td><td>73</td><td>1445</td><td>8</td><td>10</td><td>5</td><td>4</td><td>3</td><td>3</td></tr><tr><td>getmixed</td><td>74</td><td>196</td><td>103</td><td>72</td><td>2022</td><td>69</td><td>12</td><td>19</td><td>16</td><td>11</td><td>9</td></tr></tbody></table>

![](https://raw.githubusercontent.com/lslz627/PicGo/master/hdd-nofsync-time.png)

耗时基本和吞吐负相关，耗时越长，性能越差，唯一的例外是 goleveldb, 它的耗时也很短，但是吞吐率却没有足够优秀。

### 同步写盘 (fsync)

**throughputs**

<table><thead><tr><th></th><th>badger</th><th>bbolt</th><th>bolt</th><th>leveldb</th><th>kv</th><th>buntdb</th><th>rocksdb</th><th>btree</th><th>btree/memory</th><th>map</th><th>map/memory</th></tr></thead><tbody><tr><td>set</td><td>1071</td><td>67</td><td>70</td><td>1466</td><td>5084</td><td>66</td><td>1321</td><td>66</td><td>435274</td><td>67</td><td>660196</td></tr><tr><td>del</td><td>1309</td><td>81</td><td>73</td><td>1490</td><td>7287</td><td>162</td><td>1386</td><td>164</td><td>737082</td><td>163</td><td>802890</td></tr><tr><td>get</td><td>401010</td><td>276105</td><td>323697</td><td>510568</td><td>5664</td><td>1728309</td><td>1529519</td><td>1973164</td><td>2735229</td><td>2673796</td><td>3229974</td></tr><tr><td>setmixed</td><td>64</td><td>60</td><td>31</td><td>54</td><td>124</td><td>66</td><td>-1</td><td>67</td><td>20827</td><td>66</td><td>23213</td></tr><tr><td>getmixed</td><td>4201</td><td>300291</td><td>315338</td><td>541976</td><td>4938</td><td>3082</td><td>1961553</td><td>2144</td><td>890075</td><td>2816</td><td>1406865</td></tr></tbody></table>

![](https://raw.githubusercontent.com/lslz627/PicGo/master/hdd-fsync-throughputs.png)

在数据落盘的情况下，写的性能急剧下降，因为每次写都需要同步到硬盘中，比较好的是 rocksdb 和 badger，能达到 1000+ op/s, 其它基本都在 100 以内。

**time (latency)**

<table><thead><tr><th></th><th>badger</th><th>bbolt</th><th>bolt</th><th>leveldb</th><th>kv</th><th>buntdb</th><th>rocksdb</th><th>btree</th><th>btree/memory</th><th>map</th><th>map/memory</th></tr></thead><tbody><tr><td>getmixed</td><td>5950</td><td>83</td><td>79</td><td>46</td><td>5062</td><td>8110</td><td>12</td><td>11655</td><td>28</td><td>8874</td><td>17</td></tr><tr><td>del</td><td>19089</td><td>307542</td><td>337844</td><td>16776</td><td>3430</td><td>154249</td><td>18037</td><td>151850</td><td>33</td><td>152854</td><td>31</td></tr><tr><td>setmixed</td><td>15557829</td><td>16650544</td><td>31712503</td><td>18451317</td><td>8035914</td><td>14950086</td><td>-1</td><td>14848393</td><td>48016</td><td>14978524</td><td>43080</td></tr><tr><td>set</td><td>23325</td><td>373100</td><td>352715</td><td>17043</td><td>4917</td><td>373841</td><td>18922</td><td>373197</td><td>57</td><td>372896</td><td>37</td></tr><tr><td>get</td><td>62</td><td>90</td><td>77</td><td>48</td><td>4413</td><td>14</td><td>16</td><td>12</td><td>9</td><td>9</td><td>7</td></tr></tbody></table>

![](https://raw.githubusercontent.com/lslz627/PicGo/master/hdd-nofsync-time.png)

和吞吐率负相关，goleveldb 例外。bbolt 和 kv 删除的时候也很慢。

SSD 固态硬盘
--------

采用固态硬盘，我们期望写的性能能提升起来，看测试结果。

10 块 SSD, 型号:

```
Vendor:               AVAGO

Product:              AVAGO

Revision:             4.66

User Capacity:        298,999,349,248 bytes [298 GB]

Device Speed: 6.0Gb/s

Link Speed: 12.0Gb/s

Media Type: Solid State Device
```

### 不写盘 (nofsync)

**throughputs**

<table><thead><tr><th></th><th>badger</th><th>bbolt</th><th>bolt</th><th>leveldb</th><th>kv</th><th>buntdb</th><th>rocksdb</th><th>btree</th><th>btree/memory</th><th>map</th><th>map/memory</th></tr></thead><tbody><tr><td>del</td><td>83427</td><td>3513</td><td>2922</td><td>164755</td><td>7042</td><td>187054</td><td>611694</td><td>359934</td><td>787141</td><td>617882</td><td>1190171</td></tr><tr><td>set</td><td>76760</td><td>12690</td><td>13329</td><td>128174</td><td>3683</td><td>109943</td><td>463219</td><td>185225</td><td>633252</td><td>215862</td><td>833461</td></tr><tr><td>get</td><td>454835</td><td>405501</td><td>435838</td><td>404430</td><td>17736</td><td>3054694</td><td>1467525</td><td>3555188</td><td>4150152</td><td>4706103</td><td>4747594</td></tr><tr><td>setmixed</td><td>4566</td><td>8324</td><td>9083</td><td>17123</td><td>700</td><td>14128</td><td>145839</td><td>29296</td><td>40860</td><td>50836</td><td>49103</td></tr><tr><td>getmixed</td><td>395796</td><td>72929</td><td>234612</td><td>264450</td><td>13987</td><td>282241</td><td>1889387</td><td>640031</td><td>1014941</td><td>1548680</td><td>1973040</td></tr></tbody></table>

![](https://raw.githubusercontent.com/lslz627/PicGo/master/ssd-nofsync-throughputs.png)

读写都很快，rocksdb 表现依然很优秀。而 buntdb 的只读依然很快。因为没有同步落盘的操作。

这个数据和前面的不写盘的数据不相同，是因为我换了一台有 ssd 的机器进行测试。

**time (latency)**

<table><thead><tr><th></th><th>badger</th><th>bbolt</th><th>bolt</th><th>leveldb</th><th>kv</th><th>buntdb</th><th>rocksdb</th><th>btree</th><th>btree/memory</th><th>map</th><th>map/memory</th></tr></thead><tbody><tr><td>set</td><td>651</td><td>3939</td><td>3751</td><td>390</td><td>13573</td><td>454</td><td>107</td><td>269</td><td>78</td><td>231</td><td>59</td></tr><tr><td>get</td><td>109</td><td>123</td><td>114</td><td>123</td><td>2819</td><td>16</td><td>34</td><td>14</td><td>12</td><td>10</td><td>10</td></tr><tr><td>setmixed</td><td>218995</td><td>120129</td><td>110092</td><td>58398</td><td>1427908</td><td>70777</td><td>6856</td><td>34134</td><td>24473</td><td>19670</td><td>20365</td></tr><tr><td>getmixed</td><td>126</td><td>685</td><td>213</td><td>189</td><td>3574</td><td>177</td><td>26</td><td>78</td><td>49</td><td>32</td><td>25</td></tr><tr><td>del</td><td>599</td><td>14232</td><td>17107</td><td>303</td><td>7100</td><td>267</td><td>81</td><td>138</td><td>63</td><td>80</td><td>42</td></tr></tbody></table>

![](https://raw.githubusercontent.com/lslz627/PicGo/master/ssd-nofsync-time.png)

### 同步写盘 (fsync)

**throughputs**

<table><thead><tr><th></th><th>badger</th><th>bbolt</th><th>bolt</th><th>leveldb</th><th>kv</th><th>buntdb</th><th>rocksdb</th><th>btree</th><th>btree/memory</th><th>map</th><th>map/memory</th></tr></thead><tbody><tr><td>set</td><td>32124</td><td>5413</td><td>5623</td><td>56968</td><td>5123</td><td>7052</td><td>52700</td><td>8065</td><td>614477</td><td>7841</td><td>913575</td></tr><tr><td>get</td><td>439483</td><td>336439</td><td>291307</td><td>635404</td><td>5330</td><td>1982160</td><td>3246753</td><td>3145643</td><td>3512469</td><td>3852080</td><td>3629764</td></tr><tr><td>getmixed</td><td>129789</td><td>222806</td><td>252334</td><td>609681</td><td>4013</td><td>86169</td><td>3923107</td><td>102087</td><td>850051</td><td>120716</td><td>1495438</td></tr><tr><td>setmixed</td><td>4893</td><td>4946</td><td>4390</td><td>5913</td><td>203</td><td>5083</td><td>6276</td><td>6033</td><td>32896</td><td>7291</td><td>63406</td></tr><tr><td>del</td><td>38229</td><td>6990</td><td>5732</td><td>66996</td><td>7637</td><td>18200</td><td>65530</td><td>17635</td><td>868734</td><td>17363</td><td>956571</td></tr></tbody></table>

![](https://raw.githubusercontent.com/lslz627/PicGo/master/ssd-fsync-throughputs.png)

在落盘的情况下，写操作有了大幅的提升。尤其是 goleveldb 和 badger，但是 rocksdb 的读是远远领先于其它 K/V 数据的 (除了 btree 和 map)。

**time (latency)**

<table><thead><tr><th></th><th>badger</th><th>bbolt</th><th>bolt</th><th>leveldb</th><th>kv</th><th>buntdb</th><th>rocksdb</th><th>btree</th><th>btree/memory</th><th>map</th><th>map/memory</th></tr></thead><tbody><tr><td>set</td><td>1556</td><td>9236</td><td>8890</td><td>877</td><td>9758</td><td>7089</td><td>948</td><td>6199</td><td>81</td><td>6376</td><td>54</td></tr><tr><td>get</td><td>113</td><td>148</td><td>171</td><td>78</td><td>9379</td><td>25</td><td>15</td><td>15</td><td>14</td><td>12</td><td>13</td></tr><tr><td>setmixed</td><td>204372</td><td>202173</td><td>227763</td><td>169098</td><td>4914902</td><td>196697</td><td>159317</td><td>165745</td><td>30399</td><td>137151</td><td>15771</td></tr><tr><td>getmixed</td><td>385</td><td>224</td><td>198</td><td>82</td><td>12459</td><td>580</td><td>12</td><td>489</td><td>58</td><td>414</td><td>33</td></tr><tr><td>del</td><td>1307</td><td>7152</td><td>8721</td><td>746</td><td>6546</td><td>2747</td><td>763</td><td>2835</td><td>57</td><td>2879</td><td>52</td></tr></tbody></table>

![](https://raw.githubusercontent.com/lslz627/PicGo/master/ssd-fsync-throughputs.png)

总的来说，rocksdb 的表现不错。为了保证不丢数据，我们可能需要设置同步硬盘的参数，但是这可能影响写的性能，需要通过批量写和 SSD 来解决。  
对 badger 有点小失望，当然它对 SSD 的优化还是很可以的，有可能测试大的数据的时候才能显示出来它的优势。  
对于简单的场景，也可以采用 btree、map 这种简单的数据结构来实现，加上 AOF, 如果想减少 AOF 的大小，可以像 redis 一样合并 AOF 的操作，去掉无用的中间数据。

[**Newer**

[译] 使用 Go 语言读写 Redis 协议

](https://colobu.com/2019/04/16/Reading-and-Writing-Redis-Protocol-in-Go/)[**Older**

百万 Go TCP 连接的思考 3: 正常连接下的吞吐率和延迟

](https://colobu.com/2019/02/28/1m-go-tcp-connection-3/)