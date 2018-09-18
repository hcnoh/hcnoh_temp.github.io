---
layout: post
title: "WAY 9. 컴프리헨션이 클 때는 제너레이터 표현식을 고려하자"
date: 2018-09-17 20:24:26
image: '/assets/img/'
description: '이펙티브 파이썬 코딩의 기술 책 스터디 정리'
tags:
- python
categories:
- 이펙티브 파이썬 스터디
twitter_text: 'WAY 9. 컴프리헨션이 클 때는 제너레이터 표현식을 고려하자'
---

## 리스트 컴프리헨션의 문제점
- 입력 시퀀스에 있는 각 값별로 아이템을 하나씩 담은 새 리스트를 통째로 생성한다는 점
- 따라서 메모리를 많이 소모할 수 있음
- 예제
{% highlight python %}
value = [len(x) for x in open("/tmp/my_file.txt")]
print(value)

>>>
[100, 57, 15, 1, 12, 75, 5, 86, 89, 11]
{% endhighlight %}
- 위의 예제에서는 리스트 컴프리헨션으로 인하여 파일에 있는 각 줄의 길이만큼 메모리가 필요함
  - 즉, 위의 예제에서 `value`를 생성할 때 차지하는 메모리를 정리하면:
    - `open`으로 열었던 txt 파일
    - `x in open()`으로 생성되는 소스 리스트 (value가 타겟 리스트)
  - 파일에 오류가 있거나 끊김이 없는 네트워크 소켓일 경우 문제가 발생

## 제너레이터 표현식
- 파이썬은 이러한 리스트 컴프리헨션의 문제를 해결하기위해 제너레이터 표현식을 제공
- 제너레이터 표현식?
  - 리스트 컴프리헨션과 제너레이터를 일반화한 것
  - 실행될 때 출력 시퀀스를 모두 구체화(메모리에 로딩)하지 않음
  - 표현식에서 한 번에 한 아이템을 내주는 이터레이터 기능
- 위의 예제와 동치인 제너레이터 표현식 예제
{% highlight python %}
it = (len(x) for x in open("/tmp/my_file.txt"))
print(it)

>>>
<generator object <genexpr> at 0x101b81480>
{% endhighlight %}
- 위의 예제에서는 메모리에 `open`으로 열었던 txt 파일만이 올라감
- 필요한 경우 `next` 사용: `next` 이후 이터레이터는 전진
{% highlight python %}
print(next(it))
print(next(it))
  
>>>
100
57
{% endhighlight %}

## 다른 제너레이터 표현식과 연계
- 예제
{% highlight python %}
it = (len(x) for x in open("/tmp/my_file.txt"))
roots = ((x, x**0.5) for x in it)
print(next(roots))

>>>
(15, 3.872983346207417)
{% endhighlight %}
- 위의 예제에서 `roots`를 전진시키면 루프의 도미노 효과로 내부 이터레이터인 `it`도 함께 전진

## 주의 사항
- 제너레이터 표현식이 반환한 이터레이터에는 상태가 존재
- 따라서 이터레이터를 한 번 넘게 사용하지 않도록 주의
- "WAY 17. 인수를 순회할 때는 방어적으로 하자" 