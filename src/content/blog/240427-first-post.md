---
title: "nest if-else剋星-Guard Clause"
description: "分享一個簡單的技巧，讓你避免使用過多的if-else"
pubDate: "2024-04-27"
heroImage: "/guard.webp"
---

**前言**

在程式開發中，我們常常會遇到需要判斷條件，然後執行不同的程式碼。這時候我們會使用if-else來處理，但是過多的if-else會讓程式碼變得複雜，難以維護。這時候我們可以使用Guard Clause來取代if-else，讓程式碼更簡潔易讀。

**Guard Clause是什麼？**

Guard Clause是一種程式碼撰寫技巧，用來處理邊界條件。當條件不符合時，直接返回，避免進入巢狀if-else的判斷流程。

**範例**

```javascript
function nestedIfElse() {
  if (conditionA) {
    if (conditionB) {
      if (conditionC) {
        console.log("conditionC is true");
      } else {
        console.log("conditionC is false");
      }
    } else {
      console.log("conditionB is false");
    }
  } else {
    console.log("conditionA is false");
  }
}
```

可以看到上面的程式碼有三層的巢狀if-else，讓程式碼變得複雜。我們可以使用Guard Clause來簡化程式碼。

```javascript
function guardClause() {
  if (!conditionA) {
    console.log("conditionA is false");
    return;
  }

  if (!conditionB) {
    console.log("conditionB is false");
    return;
  }

  if (!conditionC) {
    console.log("conditionC is false");
    return;
  }

  console.log("conditionC is true");
}
```

可以看到使用Guard Clause後，程式碼變得更簡潔易讀。當條件不符合時，直接返回，避免進入巢狀if-else的判斷流程。

**結論**

Guard Clause是一種程式碼撰寫技巧，用來處理邊界條件。當條件不符合時，直接返回，避免進入巢狀if-else的判斷流程。這樣可以讓程式碼更簡潔易讀，減少錯誤發生的機會。希望這個技巧對你有所幫助，謝謝！
