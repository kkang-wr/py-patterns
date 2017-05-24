# 行为型模式

- [观察者模式（Observer）](#观察者模式)
- [策略模式（Strategy）](#策略模式)
- [命令模式（Command）](#命令模式)
- [模板方法模式（Template Method）](#模板方法模式)
- [迭代器模式（Itertor）](#迭代器模式)
- [状态模式（State）](#状态模式)

## 观察者模式

### 定义

定义对象间的一种一对多的依赖关系 ,当一个对象的状态发生改变时 , 所有依赖于它的对象都得到通知并被自动更新。

### 动机

将一个系统分割成一系列相互协作的类有一个常见的副作用:需要维护相关对象间的一致性。我们不希望为了维持一致性而使各类紧密耦合，因为这样降低了它们的可重用性。

### 适用性

- 当一个抽象模型有两个方面 , 其中一个方面依赖于另一方面。将这二者封装在独立的对象中以使它们可以各自独立地改变和复用。
- 当对一个对象的改变需要同时改变其它对象 , 而不知道具体有多少对象有待改变。
- 当一个对象必须通知其它对象，而它又不能假定其它对象是谁。换言之 , 你不希望这些对象是紧密耦合的。

### 优缺点

- 目标和观察者间的抽象耦合
- 支持广播通信
- 意外的更新

### 实现

> 有一个气象站可以获取温度、湿度、氧气的数据，和一些面板，每当数据更新时候要显示在面板上 —— 《Head First 设计模式》

```python
class AbstractObservable(object):
    def register(self):
        raise NotImplementedError(
            'register is a abstract method which must be implemente')

    def remove(self):
        raise NotImplementedError(
            'remove is a abstract method which must be implemente')
```

观察者这观察的对象，称作可观察对象，该抽象类需要实现具体的注册和删除观察者管理方法。

```python
class AbstractDisplay(object):
    def update(self):
        raise NotImplementedError(
            'update is a abstract method which must be implemente')

    def display(self):
        raise NotImplementedError(
            'display is a abstract method which must be implemente')
```

观察者抽象类，需要实现 update 方法，让可观察对象可以通知观察者。

```python
class Subject(object):
    def __init__(self, subject):
        self.subject = subject
        self._observers = []

    def register(self, ob):
        self._observers.append(ob)

    def remove(self, ob):
        self._observers.remove(ob)

    def notify(self, data=None):
        for ob in self._observers:
            ob.update(data)
```

此外还实现了一个 Subject 用于管理多个事件的通知，可以称作可观察对象管理者。

```python
class WeatherData(AbstractObservable):
    def __init__(self, *namespaces):
        self._nss = {}
        self._clock = None
        self._temperature = None
        self._humidity = None
        self._oxygen = None

        for ns in namespaces:
            self._nss[ns] = Subject(ns)

    def register(self, ns, ob):
        if ns not in self._nss:
            raise Exception('this {} is invalid namespace'.format(ns))
        self._nss[ns].register(ob)

    def remove(self, ns, ob):
        return self._nss[ns].remove(ob)

    def set_measurement(self, data):
        # 此处实现可以更加紧凑，但是为了表达更简单，采用如下方式
        self._clock = data['clock']
        self._temperature = data['temperature']
        self._humidity = data['humidity']
        self._oxygen = data['oxygen']

        for k in self._nss.keys():
            if k != 'all':
                data = self

            self._nss[k].notify(data)
	
    # 以下 property 为了实现 pull 模式
    
    @property
    def clock(self):
        return self._clock

    @property
    def temperature(self):
        return self._temperature

    @property
    def humidity(self):
        return self._humidity
    @property
    def oxygen(self):
        return self._oxygen

```

观察者模式的可观察对象实现可以分成两种实现方案：

- push 模式
- pull 模式

push 模式能保证所有的观察者可以接收到全部的数据，无论需要不需要，频繁更新会影响性能。

pull 模式需要观察者自己拉去数据，实现起来比较容易出错，但是能按需获取信息。

```python
class OverviewDisplay(AbstractDisplay):
    def __init__(self):
        self._data = {}

    def update(self, data):
        self._data = data
        self.display()

    def display(self):
        print(u'总览显示面板：')
        for k, v in self._data.items():
            print(k + ': ' + str(v))
```

这是一个总览的 Display ，采用 push 模式更新，获取当前能获取的所有数据，并且显示出来。

```python
class TemperatureDisplay(AbstractDisplay):
    def __init__(self):
        self._storage = []

    def update(self, data):
        dt = data.clock
        temperature = data.temperature
        self._storage.append((dt, temperature))
        self.display()

    def display(self):
        print(u'温度显示面板：')
        for storey in self._storage:
            print(storey[0] + ': ' + str(storey[1]))
```

一个只会显示温度的 Display，能观察到时间和温度变化，由于只关心温度数据，所以采用 pull 模式更加合适。

```python
if __name__ == '__main__':
    import time
    
    # 生成一个可观察对象，支持('all', 'temperature', 'humidity', 'oxygen')的数据通知
    wd = WeatherData('all', 'temperature', 'humidity', 'oxygen')
    
    # 两个观察者对象
    od = OverviewDisplay()
    td = TemperatureDisplay()

    # 注册到可观察对象中，能获取数据更新
    wd.register('all', od)
    wd.register('temperature', td)
	
    # 更新数据，可观察对象将会自动更新数据
    wd.set_measurement({
        'clock': time.strftime("%Y-%m-%d %X", time.localtime()),
        'temperature': 20,
        'humidity': 60,
        'oxygen': 10
    })
	
    # 一秒后再次更新数据
    time.sleep(1)
    print('\n')
    wd.set_measurement({
        'clock': time.strftime("%Y-%m-%d %X", time.localtime()),
        'temperature': 21,
        'humidity': 58,
        'oxygen': 7
    })
```

执行的结果如下：

```shell
总览显示面板：
humidity: 60
temperature: 20
oxygen: 10
clock: 2017-03-26 18:08:41
温度显示面板：
2017-03-26 18:08:41: 20

总览显示面板：
humidity: 58
temperature: 21
oxygen: 7
clock: 2017-03-26 18:08:42
温度显示面板：
2017-03-26 18:08:41: 20
2017-03-26 18:08:42: 21
```

一秒后数据更新，两个面板会自动更新数据。

> Python 设计模式相关代码可以 https://github.com/zhengxiaowai/design-patterns 获得。
>
> 该模式的代码可以从 https://raw.githubusercontent.com/zhengxiaowai/design-patterns/master/behavioral/observer.py 获得

当需要一个湿度面板时候也是只需要生成这个面板、并且实现你所需要的 update、display 方法然后再注册到可观察对象中即可，无须修改其他部分，实现了结构的解耦。

观察者模式在很多软件和框架中经常出现，比如 MVC 框架，事件的循环等应用场景。若希望在一个对象的状态变化时能够通知/提醒所有相关者（一个对象或一组对象），则可以使用观察者模式。观察者模式的一个重要特性是，在运行时，订阅者/观察者的数量以及观察者是谁可能会变化，也可以改变。

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

## 命令模式

### 定义

将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化;对请求排队或记录请求日志，以及支持可撤消的操作。

### 动机

有时必须向某对象提交请求，但并不知道关于被请求的操作或请求的接受者的任何信息。

### 适用性

- 抽象出待执行的动作参数化某对象（调用者和实现者解耦）。
- 在不同时候制定、排列和执行请求。
- 支持取消操作。
- 支持修改日志，这样当系统崩溃时，这些修改可以被重做一遍。
- 用构建在原语操作上的高层操作构造一个系统（事物）。

### 实现

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-


class Transaction(object):
    def __init__(self):
        self.commands = []
        self.success = []

    def add_command(self, command):
        self.commands.append(command)

    def excute(self):
        """ 调用者不需要知道执行什么，只知道有 excute 方法"""
        for command in self.commands:
            command.excute()
            self.success.append(command)

    def undo(self):
        for command in self.success[::-1]:
            command.undo()


class CreateCommand(object):
    def __init__(self, filename):
        self.filename = filename

    def excute(self):
        print('create a {}'.format(self.filename))

    def undo(self):
        print('delete this {}'.format(self.filename))


class WriteCommand(object):
    def __init__(self, filename, content):
        self.filename = filename
        self.content = content

    def excute(self):
        print('write [{}] to {}'.format(self.content, self.filename))

    def undo(self):
        print('remove [{}] from {}'.format(self.content, self.filename))


class ChomdCommand(object):
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode

    def excute(self):
        print('change {} mode to {}'.format(self.filename, self.mode))

    def undo(self):
        print('revocer {} mode to {}'.format(self.filename, '644'))


class MoveCommand(object):
    """ 假设这个命令发生了错误 """

    def __init__(self, filename, to_path):
        self.filename = filename
        self.to_path = to_path

    def excute(self):
        print('move {} to {}'.format(self.filename, self.to_path))
        raise Exception('you have not permission')

    def undo(self):
        print('move {} to {}'.format(self.to_path, self.filename))


if __name__ == '__main__':
    create_command = CreateCommand('test.file')
    write_command = WriteCommand('test.file', 'my name is zhengxiaowai')
    chmod_command = ChomdCommand('test.file', '600')

    file_operation = Transaction()
    file_operation.add_command(create_command)
    file_operation.add_command(write_command)
    file_operation.add_command(chmod_command)

    # file_operation.excute()

    try:
        # 发生错误恢复原始状态
        move_command = MoveCommand('test.file', '/etc/')
        file_operation.add_command(move_command)
        file_operation.excute()
    except:
        print('\nraise a error, start to undo:\n')
        file_operation.undo()
```

## 模板方法模式

### 定义

在一个方法中定义一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。

### 动机

设计一个应用框架，这个框架按照固定的流程实现，但是某些方法需要子类来提供。也可以提供钩子，让子类来决定做什么。

### 适用性

- 一次性实现一个算法的不变部分，将可变的行为留给子类实现。
- 各子类中公共方法的行为应被提取出来并集中到一个公共父类中以避免代码重复。
- 控制子类扩展，在特定的点调用钩子操作，只允许在这些点进行扩展。

### 实现

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from abc import ABCMeta, abstractmethod


class CaffeineBeverage(metaclass=ABCMeta):
    def make_beverage(self):
        self.boil_water()
        self.brew()
        self.pour_in_cup()
        if (self.customer_want_condiments()):
            self.add_condiments()

    def boil_water(self):
        print('把水烧开')

    def pour_in_cup(self):
        print('倒入杯子中')

    def customer_want_condiments(self):
        # hook
        return False

    @abstractmethod
    def brew(self):
        pass

    @abstractmethod
    def add_condiments(self):
        pass


class Tea(CaffeineBeverage):
    def brew(self):
        print('用沸水浸泡茶叶')

    def customer_want_condiments(self):
        return True

    def add_condiments(self):
        print('添加柠檬')


class Coffee(CaffeineBeverage):
    def brew(self):
        print('用沸水冲泡咖啡')

    def customer_want_condiments(self):
        return False

    def add_condiments(self):
        print('添加牛奶和糖')


if __name__ == '__main__':
    print('制作咖啡：')
    coffee = Coffee()
    coffee.make_beverage()
    print('\n制作茶：')
    tea = Tea()
    tea.make_beverage()
```

## 迭代器模式

### 定义

提供一种方法顺序访问一个聚合对象中各个元素 , 而又不需暴露该对象的内部表示。

### 动机

不同的对象有不同的遍历方式，即使可以 预见到所需的那些遍历操作，你可能也不希望列表的接口中充斥着各种不同遍历的操作。
有时还可能需要在同一个表列上同时进行多个遍历。

### 适用性

- 访问一个聚合对象的内容而无需暴露它的内部表示。
- 支持对聚合对象的多种遍历。
- 为遍历不同的聚合结构提供一个统一的接口 (即, 支持多态迭代)。

### 实现

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-


class BreakfastMenu(object):
    def __init__(self):
        self.items = []

    def add_item(self, name, price):
        self.items.append((name, price))

    def __iter__(self):
        """ return a Iterable object """
        return iter(self.items)


class LaunchMenu(object):
    def __init__(self):
        self.items = set()

    def add_item(self, name, price):
        self.items.add((name, price))

    def __iter__(self):
        """ return a Iterable object """
        return iter(self.items)


class DinnerMenu(object):
    def __init__(self):
        self.items = {}

    def add_item(self, name, price):
        self.items[name] = price

    def __iter__(self):
        """ return a Iterable object """
        return iter(((name, price) for name, price in self.items.items()))


if __name__ == '__main__':
    breakfast_menu = BreakfastMenu()
    breakfast_menu.add_item('milk', 5)
    breakfast_menu.add_item('bread', 6)
    breakfast_menu.add_item('coffee', 7)
    breakfast_menu.add_item('donuts', 3)

    print('\nBreakfastMenu:')
    for item in breakfast_menu:
        print(item)

    launch_menu = LaunchMenu()
    launch_menu.add_item('milk', 5)
    launch_menu.add_item('bread', 6)
    launch_menu.add_item('coffee', 7)
    launch_menu.add_item('donuts', 3)

    print('\nLaunchMenu:')
    for item in launch_menu:
        print(item)

    dinner_menu = DinnerMenu()
    dinner_menu.add_item('milk', 5)
    dinner_menu.add_item('bread', 6)
    dinner_menu.add_item('coffee', 7)
    dinner_menu.add_item('donuts', 3)

    print('\nDinnerMenu:')
    for item in dinner_menu:
        print(item)
```

## 状态模式

### 定义

允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。

### 动机

事物存在有限的不同状态，并且可以在状态中互相切换，作为相应的行为。

### 适用性

- 一个对象的行为取决于它的状态,并且它必须在运行时刻根据状态改变它的行为。
- 一个操作中含有庞大的多分支的条件语句，且这些分支依赖于该对象的状态。

### 实现

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-


class State(object):
    def __init__(self, process):
        self.process = process

    def execute(self):
        raise NotImplementedError('you must implemment this method')

    def handup(self):
        raise NotImplementedError('you must implemment this method')

    def killed(self):
        raise NotImplementedError('you must implemment this method')


class ProcessError(Exception):
    pass


class CreatedState(State):
    def execute(self):
        print('execute process, transform to running state')
        self.process.set_state(self.process.running_state)

    def handup(self):
        print('handup process, transform to waitting state')
        self.process.set_state(self.process.waitting_state)

    def killed(self):
        print('killed process, transform to waitting teminated')
        self.process.set_state(self.process.terminated_state)


class WaittingState(State):
    def execute(self):
        print('execute process, transform to running state')
        self.process.set_state(self.process.running_state)

    def handup(self):
        print('handup process, transform to waitting state')
        self.process.set_state(self.process.waitting_state)

    def killed(self):
        print('killed process, transform to waitting teminated')
        self.process.set_state(self.process.terminated_state)


class RunningState(State):
    def execute(self):
        raise ProcessError('running state have not execute method')

    def handup(self):
        raise ProcessError('running state have not handup method')

    def killed(self):
        print('killed process, transform to waitting teminated')
        self.process.set_state(self.process.terminated_state)


class TerminateState(State):
    def execute(self):
        raise ProcessError('running state have not execute method')

    def handup(self):
        raise ProcessError('running state have not handup method')

    def killed(self):
        print('process was teminated')
        self.process.set_state(self.process.terminated_state)


class Process(object):
    def __init__(self, name):
        self.created_state = CreatedState(self)
        self.waitting_state = WaittingState(self)
        self.running_state = RunningState(self)
        self.terminated_state = TerminateState(self)

        self.name = name
        self.state = self.created_state

    def __str__(self):
        return self.state.__class__.__name__

    def set_state(self, state):
        self.state = state

    def execute(self):
        self.state.execute()

    def handup(self):
        self.state.handup()

    def killed(self):
        self.state.killed()


if __name__ == '__main__':
    print('Process NO.1')
    p1 = Process('p1')
    p1.execute()
    p1.killed()

    print('\nProcess NO.2')
    p2 = Process('p2')
    p2.handup()
    p2.execute()
    p2.killed()

    print('\nProcess NO.3')
    p3 = Process('p3')
    p3.handup()
    p3.handup()
    p3.killed()

    print('\nProcess NO.4')
    p4 = Process('p4')
    p4.killed()
```

