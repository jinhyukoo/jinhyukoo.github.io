---
title: HackerRank Pairs
layout: post
date: '2021-01-13 01:44:00'
author: 진혀크
tags: HackerRank Pairs Binary search 이분탐색
cover: "/assets/instacode.png"
categories: PS
---

## Problem

[Problem Link](https://www.hackerrank.com/challenges/pairs/problem)

## Sample Input


    5 2
    1 5 3 4 2

## Code

    int pairs(int k, vector<int> arr) {
        int result = 0;
        sort(arr.begin(), arr.end());
        for(int i = 0 ; i<arr.size() ; i++){
            int left = i, right = arr.size() - 1;
            while(left<=right){
                int mid = (left + right) / 2;
                int now = arr[mid];
                if(now - arr[i] == k){
                    result++;
                    break;
                }
                else if(now - arr[i] > k) right = mid - 1;
                else left = mid + 1;
            }
        }
        return result;
    }

    