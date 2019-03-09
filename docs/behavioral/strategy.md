## 策略模式

### 定义

定义一系列算法，把他们封装起来，并且可以相互替换使用，使得算法可以独立于客户端而变化。

### 适用性

- 许多相关类仅仅只有算法不同
- 需要使用使用一个算法的不用变体
- 算法使用客户不应该知道的数据
- 一个类定义了多种行为 

### 优缺点

- 一系列可重用的算法或者行为
- 一种替代继承的方法
- 消除一些条件语句
- 可以选择不同的行为或者算法
- 客户端要知道策略的不同地方

### 实现

> 年底了，需要发年终奖，年终奖需要根据本年度的绩效来决定拿多少，绩效的等级有 S、A、B 三种。
> 实现一个计算每个人年终奖的程序，后期可能会添加 S+、C 等级或者更多。

#### Python Class

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

# 定义一系列计算年终奖的算法
bouns_strategy = {
    'S': lambda s: s * 4,
    'A': lambda s: s * 3,
    'B': lambda s: s * 2,

    # 添加 S+ 和 C 算法
    'SP': lambda s: s * 5,
    'C': lambda s: s ,
}


class Bouns:
    def __init__(self, name, salary):
        self.name = name
        self.salary = salary
        # 计算方法通过属性来设定，这样可以容易被替换
        self.calculate = None

    def __str__(self):
        return "{} get {}".format(self.name, self.bouns)

    @property
    def bouns(self):
        return self.calculate(self.salary)


if __name__ == '__main__':
    jack_bouns = Bouns('jack', 10000)
    jack_bouns.calculate = bouns_strategy['S']
    print(jack_bouns)

    linda_bouns = Bouns('linda', 10000)
    linda_bouns.calculate = bouns_strategy['A']
    print(linda_bouns)

    sean_bouns = Bouns('sean', 10000)
    sean_bouns.calculate = bouns_strategy['B']
    print(sean_bouns)

    # 现在需求变化了，添加 S+ 作为最高级，C 级 作为最低级
    # jack 从 S 调整到 S+
    # sean 从 B 调整到 C
    print('需求改变以后......')

    jack_bouns.calculate = bouns_strategy['SP']
    print(jack_bouns)

    print(linda_bouns)

    sean_bouns.calculate = bouns_strategy['C']
    print(sean_bouns)

    # 从上面的计算年终的算法可以看出
    # 需求改变以后只需要替代原来算法就可实现了
```

#### Python Function

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

# 定义一系列计算年终奖的算法
bouns_strategy = {
    'S': lambda s: s * 4,
    'A': lambda s: s * 3,
    'B': lambda s: s * 2,

    # 添加 S+ 和 C 算法
    'SP': lambda s: s * 5,
    'C': lambda s: s,
}


def calculate_bouns(name, strategy, salary):
    return '{name} get {salary}'.format(name=name, salary=strategy(salary))


if __name__ == '__main__':
    print(calculate_bouns('jack', bouns_strategy['S'], 10000))
    print(calculate_bouns('linda', bouns_strategy['A'], 10000))
    print(calculate_bouns('sean', bouns_strategy['B'], 10000))

    # 现在需求变化了，添加 S+ 作为最高级，C 级 作为最低级
    # jack 从 S 调整到 S+
    # sean 从 B 调整到 C
    print('需求改变以后......')

    print(calculate_bouns('jack', bouns_strategy['SP'], 10000))
    print(calculate_bouns('linda', bouns_strategy['A'], 10000))
    print(calculate_bouns('sean', bouns_strategy['C'], 10000))
```