# KMP
线性复杂度求出一个串是另一个串的自串

## 失效函数
该函数的目标使得 $$ b_{1}b_{2}....b_{f(s)} 即是 b_{1}b_{2}...b_{s}最长的真前缀, 又是b_{1}b_{2}...b{s}的后缀的字串$$.

算法是:

```python
def failure_function(b: str):

    str_length = len(b)
    f_index = 1
    f = [0 for i in range(str_length)]

    while f_index < str_length:

        # f[s] 利用 已经计算的结果{f[s-1], f[s-2] .... f[0]}
        reuse_index = f_index - 1
        overlap_index = f[reuse_index]
        
        while b[overlap_index] != b[f_index]:
            reuse_index = overlap_index - 1        # 遍历 {f[s-1], f[s-2]....f[0]} 
                                                   # 找到 b[x] == b[s]            
            
            if (reuse_index < 0):                  # 遍历到f[0]
                break

            overlap_index = f[reuse_index]

                                                   # b[x] == b[s]
        if (reuse_index < 0):                      # 遍历到f[0]
            f[f_index] = 0
        else:
            f[f_index] = f[reuse_index] + 1

        f_index += 1
    return f
```


关于失效函数的应用:

计算得到关键字b1b2...bn的失效函数之后， 就可以在o(m)时间内扫描字符串a1a2...am以判断该关键字是否出现在其中.

1. 不断尝试将关键字的下一个字符与被匹配字符串的下一个字符匹配， 逐步推进,  也就是 
    
    1. 令i 不断增加， 如果b[i] == a[j], i, j都增加1 
    2. 如果j == m, 则匹配上了， 返回```true```
    3. 如果匹配了s个字符后没有匹配上,  则改变j的值, 
        1. 如果j的值是0， 则不变，  i += 1
        2. 如果j的值>0, 调整j的位置到失效函数保存的位置，即 j = ```f[j - 1] ```

算法是:
```python
def kmp(a, b):
    p = failure_function(a)

    i = 0
    j = 0

    n = len(a)
    m = len(b)

    while i < n:
        if b[j] == a[i]:
            i += 1
            j += 1

        if j == m:
            return True

        elif i < n and b[j] != a[i]:
            if j != 0:
                j = p[j - 1]

            else:
                i += 1

    return False
```





