---
 layout: post
 tags: Redis
 title: Redis with Python
---
###  Redis with Python

In order to use Redis with Python, we will need a Python Redis client. 
In following sections, we will demonstrate the use of redis-py, a Redis Python Client.
```redis-py``` requires a running Redis server.

#### Redis with Python 
Use pip to install redis-py:

```shell
$ sudo pip install redis
```
To create a connection to Redis using redis-py:
```shell
$ python
Python 3.4.3 (default, Oct 14 2015, 20:28:29) 
[GCC 4.8.4] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import redis
>>> r = redis.Redis(host='localhost', port=6379, db=0)
```
The port=6379 and db=0 are default values.

#### Reading and Writing Data

Now that we are connected to Redis, we can start reading and writing data.

The following code snippet writes the ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) `value bar` to the Redis ![#1589F0](https://placehold.it/15/1589F0/000000?text=+) `key foo`, reads it back, and prints it:
```python
>>> r.set('foo','bar')
True
>>> r.get('foo')
b'bar
```
The set key to hold the string value. If key already holds a value, it is overwritten, regardless of its type. Any previous time to live associated with the key is discarded on successful SET operation.

#### incr/decr
The incr/decr increments/decrements the number stored at key by one.

If the key does not exist, it is set to 0 before performing the operation.

```python
>>> r.set('count',1)
True
>>> r.incr('count')
2
>>> r.incr('count')
3
>>> r.decr('count')
2
>>> r.get('count')
b'2'
```
Error case:

```python
>>> r.set('count',123456789012345678901234567890)
True
>>> r.incr('count')
redis.exceptions.ResponseError: value is not an integer or out of range
```
#### rpush, llen, and lindex

The **rpush** inserts all the specified values at the tail of the list stored at key.

The **llen** returns the length of the list stored at key.

The **lindex** returns the element at index index in the list stored at key. The index is zero-based, so 0 means the first element, 1 the second element and so on.
```python
>>> r.rpush('hispanic', 'uno')
1
>>> r.rpush('hispanic', 'dos')
2
>>> r.rpush('hispanic', 'tres')
3
>>> r.rpush('hispanic', 'cuatro')
4
>>> r.llen('hispanic')
4
>>> r.lindex('hispanic', 3)
b'cuatro'
```
