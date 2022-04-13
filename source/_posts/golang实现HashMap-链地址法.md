---
title: golang实现HashMap(链地址法)
date: 2022-04-13 17:25:17
tags: golang redis
categories: golang redis
---
## hash底层
## redis hash && golang map->并发安全
## rehash

```go
package main

import (
	"fmt"
	"math/rand"
	"time"
)

const (
	initSize = 1 << 4 //哈希数组初始大小
)

type HashKey interface {
	GetHash(mod int) int//哈希函数
}

type HashNode struct {
	key   HashKey
	value interface{}
	next  *HashNode
}

type HashMap struct {
	nodes []*HashNode//哈希数组
	total int //当前元素数量
}

func (h *HashMap) init() {
	h.nodes = make([]*HashNode, initSize)//初始化哈希数组
	h.total = 0
}

func NewHashMap() *HashMap {
	mp := &HashMap{}
	mp.init()
	return mp
}

func (h *HashMap) Get(key HashKey) (value interface{}, ok bool) {
	i := key.GetHash(cap(h.nodes))//计算hashcode
	for head := h.nodes[i]; head != nil; head = head.next {//遍历hashcode对应的链表
		if head.key == key {
			ok = true
			value = head.value
			break
		}
	}

	return
}

func (h *HashMap) Set(key HashKey, value interface{}) {
	i := key.GetHash(cap(h.nodes))//计算hashcode
	for head := h.nodes[i]; head != nil; head = head.next {//遍历hashcode对应的链表
		if head.key == key {
			// key存在，替换value
			head.value = value
			return
		}
	}
	// key不存在，插入
	h.nodes[i] = &HashNode{
		key:   key,
		value: value,
		next:  h.nodes[i],
	}

	h.total++
	if h.total == cap(h.nodes) {//当前元素数量等于哈希数组的容量，扩容
		h.remake()
	}
}

func (h *HashMap) remake() {
	capa := cap(h.nodes) << 1           //容量翻倍
	newNodes := make([]*HashNode, capa) //新的hash数组
	for _, v := range h.nodes {         //遍历旧hash数组
		for ; v != nil; v = v.next {
			index := v.key.GetHash(capa)//重新计算哈希值
			newNodes[index] = &HashNode{//建立新链表
				key:   v.key,
				value: v.value,
				next:  newNodes[index],
			}
		}
	}
	h.nodes = newNodes
}

type KeyType struct {
	string
}

func (h KeyType) GetHash(mod int) (hashCode int) {//实现一个对string的哈希函数
	for _, v := range h.string {
		a := int(v)
		hashCode = (hashCode*31 + a) & (mod - 1) //mod是2的幂，直接用与代替模
	}
	return
}


//以下为测试代码
func GetRandomString(l int) string {
	str := "0123456789abcdefghijklmnopqrstuvwxyz"
	bytes := []byte(str)
	result := []byte{}
	r := rand.New(rand.NewSource(time.Now().UnixNano()))
	for i := 0; i < l; i++ {
		result = append(result, bytes[r.Intn(len(bytes))])
	}
	return string(result)
}

type cases struct {
	k, v string
}

func main() {
	mp := make(map[string]interface{}, 16)
	myMp := NewHashMap()
	n := 200000

	c := make([]cases, n)
	cc := make([]KeyType, n)
	for i := 0; i < n; i++ {
		k, v := GetRandomString(10), GetRandomString(10)
		c[i] = cases{k: k, v: v}
		cc[i] = KeyType{string: k}
	}
	st1 := time.Now().Nanosecond()
	for i := 0; i < n; i++ {
		mp[c[i].k] = c[i].v
	}
	st2 := time.Now().Nanosecond()
	for i := 0; i < n; i++ {
		myMp.Set(cc[i], c[i].v)
	}
	st3 := time.Now().Nanosecond()
	//fmt.Printf("%+v", myMp)
	for i := 0; i < n; i++ {
		_, _ = mp[c[i].k]
	}
	st4 := time.Now().Nanosecond()
	for i := 0; i < n; i++ {
		_, _ = myMp.Get(cc[i])

		// if ok2 == false || v2 != v1 {	//验证正确性
		// 	fmt.Println(v1)
		// 	fmt.Println(v2)
		// 	panic("bug!!!")
		// }
	}
	st5 := time.Now().Nanosecond()
	fmt.Printf("standard map insert: %dms\n", (st2-st1)/1000000)
	fmt.Printf("my map insert: %dms\n", (st3-st2)/1000000)
	fmt.Printf("standard map find: %dms\n", (st4-st3)/1000000)
	fmt.Printf("my map find: %dms\n", (st5-st4)/1000000)

}

```