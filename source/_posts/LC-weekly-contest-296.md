---
title: LC weekly contest 296
date: 2022-06-16 17:55:27
tags: [Leetcode, 对顶栈]
categories: Leetcode周赛
katex: true
description:  LC 周赛第296场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

###  **Solution**

#### [设计一个文本编辑器](https://leetcode.cn/problems/design-a-text-editor/)

考试时用stl写了个模拟过了，后来加强了数据被卡掉了。正确做法应该是利用双链表或者对顶栈，这种构造还是比较巧妙的。

```cpp
class TextEditor {
public:

    vector<char> left, right;

    TextEditor() {
    }
    
    void addText(string text) {
        left.insert(left.end(), text.begin(), text.end());
    }  
    
    int deleteText(int k) {
        int res = 0;
        while (k && left.size()) {
            left.pop_back();
            k--;
            res++;
        }
        return res;
    }
    
    string cursorLeft(int k) {
        while (k && left.size()) {
            right.push_back(left.back());
            left.pop_back();
            k--;
        }
        return string(left.begin() + max((int)left.size() - 10, 0), left.end());
    }
    
    string cursorRight(int k) {
        while (k && right.size()) {
            left.push_back(right.back());
            right.pop_back();
            k--;
        }
        return string(left.begin() + max((int)left.size() - 10, 0), left.end());
    }
};

/**
 * Your TextEditor object will be instantiated and called as such:
 * TextEditor* obj = new TextEditor();
 * obj->addText(text);
 * int param_2 = obj->deleteText(k);
 * string param_3 = obj->cursorLeft(k);
 * string param_4 = obj->cursorRight(k);
 */
```

