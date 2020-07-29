---
title: 자바스크립트(JavaScript) 자료구조 Deque 구현해보기
layout: post
date: '2020-07-30 00:35:00'
author: 진혀쿠
tags: 자바스크립트 JavaScript 자료구조 Deque
cover: "/assets/instacode.png"
categories: JS
---
자바스크립트를 공부하면서 간단하게 구현해본 Deque 코드를 공유합니다. 

    class Deque {
        constructor() {
            this.head = null;
            this.tail = null;
            this.count = 0;
        }

        empty(){
            if(!this.count) return 1;
            else return 0;
        }

        push_front(element){
            if(this.head === null){
                this.head = element;
                this.tail = element;
            }
            else{
                let curr = this.head;
                element.next = curr;
                curr.prev = element;
                this.head = element;
            }
            this.count++;
        }

        push_back(element){
            if(this.tail === null){
                this.head = element;
                this.tail = element;
            }
            else{
                this.tail.next = element;
                element.prev = this.tail;
                this.tail = element;
            }
            this.count++;
        }

        pop_front(){
            if(!this.empty()){
                let temp = this.head;
                temp.next.prev = null;
                this.head = temp.next;
                temp = null;
                this.count--;
            }
        }

        pop_back(){
            if(!this.empty()){
                let temp = this.tail;
                temp.prev.next = null;
                this.tail = temp.prev;
                temp = null;
                this.count--;
            }
        }

        print(){
            let curr = this.head;
            while(curr){
                console.log(curr.value);
                curr = curr.next;
            }
        }

        front(){
            return this.head.value;
        }

        back(){
            return this.tail.value;
        }

        length(){
            return this.count;
        }
    }

아래는 테스트를 해보고 싶으시다면 아래와 같은 class를 선언하여 value를 설정한 뒤 element로 전달해주면 됩니다.

    class NODE{
        constructor() {
            this.value = null;
            this.next = null;
            this.prev = null;
        }
    }
