###UUID  
英文原文：https://pymotw.com/3/uuid/

用途： uuid模块实现了 RFC 4122 定义的全球唯一标示符

[RFC 4122](https://tools.ietf.org/html/rfc4122.html)定义了一个不请求中心服务便可以创建全球唯一标示符的系统。UUID 长度是128比特，像参考指导所说，“UUID能保证唯一，与时间和空间无关”。它被用于文档、域名、程序端或者其他需要唯一标示的场景。RFC 4122标准明确指出，UUID用于创建统一资源命名空间并包含下面三个算法：

1.使用IEEE 802 MAC地址作为唯一标示资源
2.使用伪随机数
3.使用已知的字符组合做Hash计算

UUID的原文组合中都会包括一个系统时间戳和一个有序的时间戳，这个有序的时间戳用于预防系统时间戳延。

Purpose:The uuid module implements Universally Unique Identifiers as described in RFC 4122.
RFC 4122 defines a system for creating universally unique identifiers for resources in a way that does not require a central registrar. UUID values are 128 bits long and, as the reference guide says, “can guarantee uniqueness across space and time.” They are useful for generating identifiers for documents, hosts, application clients, and other situations where a unique value is necessary. The RFC is specifically focused on creating a Uniform Resource Name namespace and covers three main algorithms:
Using IEEE 802 MAC addresses as a source of uniqueness
Using pseudo-random numbers
Using well-known strings combined with cryptographic hashing
In all cases, the seed value is combined with the system clock and a clock sequence value used to maintain uniqueness in case the clock is set backwards. 


####UUID 1 - IEEE 802 MAC Address  
UUID 第一个版本使用域名的MAC地址计算而来的，uuid模块调用getnode()来获取MAC地址。

UUID version 1 values are computed using the MAC address of the host. The uuid module uses getnode() to retrieve the MAC value of the current system.
```
#uuid_getnode.py
import uuid
print(hex(uuid.getnode()))
```
如果一个系统有多个网卡，因此就有多个MAC地址，那么就会得到多个返回值。

If a system has more than one network card, and so more than one MAC, any one of the values may be returned.
```
$ python3 uuid_getnode.py
0xc82a14598875
```
为了给一个host生成一个使用其MAC地址来标示的UUID,需要使用uuid1()这个函数。这个节点的标示参数是可选的，为了使用getnode()返回的值需要将这个字段设为空。

To generate a UUID for a host, identified by its MAC address, use the uuid1() function. The node identifier argument is optional; leave the field blank to use the value returned by getnode().
```
#uuid_uuid1.py
import uuid
u = uuid.uuid1()
print(u)
print(type(u))
print('bytes   :', repr(u.bytes))
print('hex     :', u.hex)
print('int     :', u.int)
print('urn     :', u.urn)
print('variant :', u.variant)
print('version :', u.version)
print('fields  :', u.fields)
print('  time_low            : ', u.time_low)
print('  time_mid            : ', u.time_mid)
print('  time_hi_version     : ', u.time_hi_version)
print('  clock_seq_hi_variant: ', u.clock_seq_hi_variant)
print('  clock_seq_low       : ', u.clock_seq_low)
print('  node                : ', u.node)
print('  time                : ', u.time)
print('  clock_seq           : ', u.clock_seq)
```
该函数返回一个UUID对象，通过该对象的只读属性可以访问其组成部分。这些属性（像hex、int、urn）是UUID的不同表现形式。

The components of the UUID object returned can be accessed through read-only instance attributes. Some attributes, such as hex, int, and urn, are different representations of the UUID value.
```
$ python3 uuid_uuid1.py
335ea282-cded-11e6-9ede-c82a14598875
<class 'uuid.UUID'>
bytes   : b'3^\xa2\x82\xcd\xed\x11\xe6\x9e\xde\xc8*\x14Y\x88u'
hex     : 335ea282cded11e69edec82a14598875
int     : 68281999803480928707202152670695098485
urn     : urn:uuid:335ea282-cded-11e6-9ede-c82a14598875
variant : specified in RFC 4122
version : 1
fields  : (861840002, 52717, 4582, 158, 222, 220083055593589)
time_low            :  861840002
time_mid            :  52717
time_hi_version     :  4582
clock_seq_hi_variant:  158
clock_seq_low       :  222
node                :  220083055593589
time                :  137023257334162050
clock_seq           :  7902
```
因为有时间戳组合进来，uuid1()每次的调用结果都不一样。

Because of the time component, each call to uuid1() returns a new value.
```
#uuid_uuid1_repeat.py
import uuid
for i in range(3):
print(uuid.uuid1())
```
在这次输出中，只有关于时间戳的数值（在uuid结果的开头）改变了

In this output, only the time component (at the beginning of the string) changes.
```
$ python3 uuid_uuid1_repeat.py
3369ab5c-cded-11e6-8d5e-c82a14598875
336eea22-cded-11e6-9943-c82a14598875
336eeb5e-cded-11e6-9e22-c82a14598875
```
因为每个电脑都有不同的MAC地址，在不同的电脑上运行同样的程序将得到完全不一样的值，下面的例子使用明确的节点ID来模拟运行在不同主机。

Because each computer has a different MAC address, running the sample program on different systems will produce entirely different values. This example passes explicit node IDs to simulate running on different hosts.
```
#uuid_uuid1_othermac.py
import uuid
for node in [0x1ec200d9e0, 0x1e5274040e]:
print(uuid.uuid1(node), hex(node))
```
除了在不同的时间点，UUID结果才会改变。

In addition to a different time value the node identifier at the end of the UUID also changes.
```
$ python3 uuid_uuid1_othermac.py
337969be-cded-11e6-97fa-001ec200d9e0 0x1ec200d9e0
3379b7e6-cded-11e6-9d72-001e5274040e 0x1e5274040e
```

####UUID 3 and 5 - Name-Based Values
在某些环境下需要将名字替换成随机数或者关联时间的标示，UUID就特别有用。V3和V5版本的UUID使用HASH算法(MD5或者SHA-1)并输入名字加上特定命名空间的组合来计算。有很多著名的被UUID预先定义的命名空间，用于支持DNS，URLS，IOS OIDS，和 知名的X.500。可以生成新的UUID值来定义新的特定程序的命名空间。

It is also useful in some contexts to create UUID values from names instead of random or time-based values. Versions 3 and 5 of the UUID specification use cryptographic hash values (MD5 or SHA-1, respectively) to combine namespace-specific seed values with names. There are several well-known namespaces, identified by pre-defined UUID values, for working with DNS, URLs, ISO OIDs, and X.500 Distinguished Names. New application-specific namespaces can be defined by generating and saving UUID values.
```
uuid_uuid3_uuid5.py
import uuid

hostnames = ['www.doughellmann.com', 'blog.doughellmann.com']
for name in hostnames:
print(name)
print('  MD5   :', uuid.uuid3(uuid.NAMESPACE_DNS, name))
print('  SHA-1 :', uuid.uuid5(uuid.NAMESPACE_DNS, name))
print()
```
使用uuid.NAMESPACE_DNS参数传递给uuid3()或者uuid5()来生成类型为DNS的UUID:

To create a UUID from a DNS name, pass uuid.NAMESPACE_DNS as the namespace argument to uuid3() or uuid5():
```
$ python3 uuid_uuid3_uuid5.py

www.doughellmann.com
MD5   : bcd02e22-68f0-3046-a512-327cca9def8f
SHA-1 : e3329b12-30b7-57c4-8117-c2cd34a87ce9

blog.doughellmann.com
MD5   : 9bdabfce-dfd6-37ab-8a3f-7f7293bcf111
SHA-1 : fa829736-7ef8-5239-9906-b4775a5abacb
```
某个命名空间下的名字对应的UUID总是一样的，不管它是在哪儿或者在什么时候计算的。

The UUID value for a given name in a namespace is always the same, no matter when or where it is calculated.
```
#uuid_uuid3_repeat.py
import uuid

namespace_types = sorted(
n
for n in dir(uuid)
if n.startswith('NAMESPACE_')
)
name = 'www.doughellmann.com'

for namespace_type in namespace_types:
print(namespace_type)
namespace_uuid = getattr(uuid, namespace_type)
print(' ', uuid.uuid3(namespace_uuid, name))
print(' ', uuid.uuid3(namespace_uuid, name))
print()
```
同一命名空间下的同样的名称对应的UUID值是==~~不一样~~==一样的。

Values for the same name in the namespaces are different.
```
$ python3 uuid_uuid3_repeat.py

NAMESPACE_DNS
bcd02e22-68f0-3046-a512-327cca9def8f
bcd02e22-68f0-3046-a512-327cca9def8f

NAMESPACE_OID
e7043ac1-4382-3c45-8271-d5c083e41723
e7043ac1-4382-3c45-8271-d5c083e41723

NAMESPACE_URL
5d0fdaa9-eafd-365e-b4d7-652500dd1208
5d0fdaa9-eafd-365e-b4d7-652500dd1208

NAMESPACE_X500
4a54d6e7-ce68-37fb-b0ba-09acc87cabb7
4a54d6e7-ce68-37fb-b0ba-09acc87cabb7
```

####UUID 4 - Random Values
有时候基于主机和命名空间计算的UUID不够不一样（有点绕口啊，意思就是有时需要不一样）。举个例子来说，UUID将用于哈希表的hash key, 多个随机的不一样的value
key为UUID，value是多个不一样的随机数的哈希表，这种情况下UUID将用于避免hash冲突。**拥有少数几个公共的数字将更容易在日志文件中找到它。［这句显得很奇怪］** 可以使用生成随机数的函数uuid4()来使UUID生成结果不一样。

Sometimes host-based and namespace-based UUID values are not “different enough.” For example, in cases where UUID is intended to be used as a hash key, a more random sequence of values with more differentiation is desirable to avoid collisions in the hash table. Having values with fewer common digits also makes it easier to find them in log files. To add greater differentiation in UUIDs, use uuid4() to generate them using random input values.
```
uuid_uuid4.py
import uuid

for i in range(3):
print(uuid.uuid4())
```
当uuid被imported的时候，随机性基于在C库有效的基础上。如果libuuid(或者 uuid.dll)能被加载并且包含生成随机数的函数，uuid模块会使用它来生成随机数。不然uuid使用os.urandom()或者其他生成随机数的模块。

The source of randomness depends on which C libraries are available when uuid is imported. If libuuid (or uuid.dll) can be loaded and it contains a function for generating random values, it is used. Otherwise os.urandom() or the random module are used.
```
$ python3 uuid_uuid4.py

7821863a-06f0-4109-9b88-59ba1ca5cc04
44846e16-4a59-4a21-8c8e-008f169c2dd5
1f3cef3c-e2bc-4877-96c8-eba43bf15bb6
```

####Working with UUID Objects
不仅仅是生成新的UUID值，它需要能够解析字符串并创建UUID对象，UUID可以轻松得执行比较和排序等操作。

In addition to generating new UUID values, it is possible to parse strings in standard formats to create UUID objects, making it easier to handle comparisons and sorting operations.
```
#uuid_uuid_objects.py
import uuid


def show(msg, l):
print(msg)
for v in l:
print(' ', v)
print()

input_values = [
'urn:uuid:f2f84497-b3bf-493a-bba9-7c68e6def80b',
'{417a5ebb-01f7-4ed5-aeac-3d56cd5037b0}',
'2115773a-5bf1-11dd-ab48-001ec200d9e0',
]

show('input_values', input_values)

uuids = [uuid.UUID(s) for s in input_values]
show('converted to uuids', uuids)

uuids.sort()
show('sorted', uuids)
```
输入的花括号像破折号一样，将会被移除。如果字符串含有前缀urn:或者uuid:,同样也会被移除。剩下的文本必须是一个16位十六进制长度的数字的文本，这才是能被识别的UUID。

Surrounding curly braces are removed from the input, as are dashes (-). If the string has a prefix containing urn: and/or uuid:, it is also removed. The remaining text must be a string of 16 hexadecimal digits, which are then interpreted as a UUID value.
```
$ python3 uuid_uuid_objects.py

input_values
urn:uuid:f2f84497-b3bf-493a-bba9-7c68e6def80b
{417a5ebb-01f7-4ed5-aeac-3d56cd5037b0}
2115773a-5bf1-11dd-ab48-001ec200d9e0

converted to uuids
f2f84497-b3bf-493a-bba9-7c68e6def80b
417a5ebb-01f7-4ed5-aeac-3d56cd5037b0
2115773a-5bf1-11dd-ab48-001ec200d9e0

sorted
2115773a-5bf1-11dd-ab48-001ec200d9e0
417a5ebb-01f7-4ed5-aeac-3d56cd5037b0
f2f84497-b3bf-493a-bba9-7c68e6def80b
```



