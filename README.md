
![](https://github.com/EagleBaby/python_module-efsm/blob/main/Platform-Python_MicroPython-blue.svg)
![](https://github.com/EagleBaby/python_module-efsm/blob/main/version-0.2.x-green.svg)
![](https://github.com/EagleBaby/python_module-efsm/blob/main/author-EagleBaby-yellowgreen.svg)
# Catalogue/ 目录
- [General/ 总体描述](#1)  
  - [State/ 状态](#11)  
  - [Processing Function/ 处理函数](#12)  
  - [add the smallest unit to fsm](#13)  
  - [config the start and end](#14)  
- [Micro Mode](#2)  
  - [how to use](#21)  
- [Standard Mode](#3)  
  - [how to use](#31)  
  - [You could jump to update other fsm](#32)  
  - [You could create a StateSet before and pass this state_set to add](#33)  
  - [By the way that StateSet is not a user object](#34)  
- [Shell Mode](#4)  
  - [how to use](#41)  
  - [but we could use decorater to make it more simple](#42)  
  - [use doc to do the same thing](#43)  
  - [use doc to a class used EfsmMeta](#44)  
- [demos](#5)  
  - [Python Demo-Efsm](#51)  
  - [MicroPython Demo-micro or StateMachine](#52)  
<span id='0'/>  

# 0 python_module-efsm  

This is module to come fsm in python.  
There are three parts of this module made to deal with different cases: micro, standard and shell  
These three modules correspond to three core.py  

这是为python设计的fsm模块。  
本模块分为三个部分，分别用于处理不同的情况：Micro, Standard, Shell  
这三个模块对应三个core.py  

install:
```python
pip install efsm
```
<span id='1'/>

# 1 General/ 总体描述
efsm is a one-stage state machine. The smallest unit is composed of several **state** and their corresponding **processing function**.

efsm是一个一段式状态机，最小单元是由若干个**状态**和其对应的**处理函数**组成的。  
<span id='11'/>

## 1.1 State/ 状态
Any Python object which can hashable could be used regard as a state. Of course, Python string object is the best choice to represent a state  
In order to create a state, you do not need to do anything more, such as:  
```python
"this is a string", (1, ), lambda :...  #  These are acceptable state objects  
```

任何可以hash的python对象都可以作为一个状态，当然，python的字符串对象是表示一个状态的最佳选择.  
为了获得一个状态，您不需要做任何操作，比如:  
```python
"this is a string", (1, ), lambda :...  #  这些都是可以接受的状态对象
```
<span id='12'/>

## 1.2 Processing Function/ 处理函数
The processing-function to execute specific tasks and return to the next state according to the current state.  
An available processing function must contain two parameters: state, o  

处理函数的作用是根据当前的状态去执行特定的函数任务以及返回下一个状态。   
一个可用的处理函数必须包含三个参数: state, o, s

### param: state:
The current state of fsm which is used to let the programmer decide which branch to exec.  
Note, however, that this function will only be called when the fsm is in a specific state, which is bound to the function in the smallest unit  

当前的fsm的状态，用于让程序员判断应该执行何种分支。   
不过请注意，这个函数只会在fsm处于特定状态时才会调用，这些特定状态是和该函数绑定在一个最小单元内的  

### param: o:
The attached running environment of the smallest unit is actually an empty object instance.  
Each smallest unit has a corresponding attached o, in which some data can be stored when the processing function is executed  

该最小单元的附带的运行环境，实际上是一个空的对象实例。  
每个最小单元都有一个与之对应的附带o, 处理函数在执行时可以将一些数据储存在这个o中.  

### param: s:
The attached running environment of the fsm is actually an empty object instance.  
Each fsm has a corresponding attached s, in which some data can be stored when the processing function is executed  

该执行该proccessing function的fsm的运行环境，实际上是一个空的对象实例。  
每个fsm都有一个与之对应的附带s, 处理函数在执行时可以将一些数据储存在这个s中.  


### return:
Jump to the next state by the return value of the processing function. If you return none, it means that there is no state transition  

通过处理函数的返回值跳转到下一个状态。如果返回None，就意味着不进行状态转移.  

```python
def update(state, o, s):  # proccessing function which only handle: idle, move, stop/ 只负责处理idle, move, stop三种状态的函数
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'
```
Then add these smallest units into the FSM object and constantly update these states to come the function of FSM
然后将这些一个个最小单元添加进fsm对象并不断更新这些状态即可实现fsm的功能
<span id='13'/>

## 1.3 add the smallest unit to fsm
use **add(...)** to add the smallest unit into your fsm
```python
def add(*state, fn=None, data=None):
  """
  add a smallest unit to your statemachine
  :param *state: include some states as a group. 
  :param fn: then assign a proccessing function to only handle this group of states
      _: None  Note that this is a must param. If you do not pass fn
  :param data: the 'o' offer to your proccessing function
      _: None  Mean it will create a empty object instance for this smallest unit.
  :return : the smallest unit.  list[tuple[*state], fn, data]
  """
```

Here are other common api:

use remove(...) to remove one state from your fsm
```python
def remove(*state, error=True):
  """
  remove a state.
  It won't remove the hole group states but only remove the state from the group states
  The smallest unit will be remove only after all of it's group states all being removed.
  # example
  .remove('idle')
  .remove('idle', 'move')

  :param *state: fsm will remove them one by one.
  :param error: bool If not find, will raise error
  :return: None
  """
```

use find(...) to find a smallest unit in fsm
```python
def find(state):
  """
  find the smallest unit corresponding to a state
  :param state: fsm find the state, and return the smallest unit
  :return: [states, fn, data] or None
  """
```

use list(...) to showcase all state-(fn, data) in fsm
```python
def list():
  """
  showcase state
  Note: it do not return list but dict
  :return: {state: (fn, data)}
  """
```

use tolist(...) to get all the smallest units in fsm
```python
def tolist():
  """
  change _groups to list
  :return: [[(*state), fn, data], ...]
  """
```

use **is_finish(...)** to get whether this fsm is finish.(only work when your fsm been setted 'end')
```python
def is_finish():
  """
  get whether the fsm is finish.
  :return: bool
  """
```

use is_prepare(...) to get whether this fsm have executed .step().
```python
def is_prepare():
  """
  get whether the fsm is steped.
  If this statemachine has link to other as target, will check whether the target is finish.
  :return: bool
  """
```

use **restart(...)** to reset your fsm but keep your config and all the smallest units
```python
def restart():
  """
  reset your fsm but keep your config and all the smallest units
  :return:
  """
```

use **step(...)** to update your fsm
```python
def step(*ex):
  """
  step, mean update once
  :param *ex: external factor interference. Before formally exec step(if not retarget to other), call these exfunc(sm)->None. If any exfunc return a non-None value will immediate stop 'step' in this time, and 'step' will return the value from that exfunc.
  :return: bool about statemachine is running-needy or not.
  """
```
<span id='14'/>

## 1.4 config the start and end
There are some attrs in your fsm:
.state   &emsp;&emsp;   # tell you the the state now fsm is. &emsp;&emsp;# default be setted as .start when first call .step W/R
.start   &emsp;&emsp;   # tell fsm what the first state is when it first .step.  &emsp;&emsp;# it is a must-fill attr W/R
.end     &emsp;&emsp;   # tell fsm should stop and will return False from .step, and when you call is_finish() will return False also. W/R
.end     &emsp;&emsp;   # tell fsm should stop and will return False from .step, and when you call is_finish() will return False also. W/R
.on_step &emsp;&emsp;   # like fn(statemachine:object)->None, will called by fsm automatic before .step return 
.on_end  &emsp;&emsp;   # like fn(statemachine:object)->None, will called by fsm automatic when state==end in .step 

At least, you must config the .start.


The statemachine usage is different in these three way:
<span id='2'/>

# 2 Micro Mode
It can deploy on micropython. The simplest fsm in this module only consist of some datum and functions. You could to use this module as a singleton object.    

该模式可以部署在micropython单片机上。这个模块只包含一些函数和数据，你可以把这个module当作一个单例对象  
<span id='21'/>

# 2.1 how to use
```python
from efsm import micro

def update(state, o, s):
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'
 
micro.add('idle', 'move', 'stop', fn=update)  # fn is must in micro mode. data is available.
micro.start = 'idle'
micro.end = 'stop'

while micro.step(): ...
print("finish.")
```
```
i'm idle, next to move
i'm moving, next to stop
finish.
```

<span id='3'/>

# 3 Standard Mode
It can deploy on micropython. This module offer a StateMachine class. And you can link their instances to a net.

该模式可以部署在micropython单片机上。这个模块提供了一个StateMachine类，并且你可以将它们的实例连接成网  
<span id='31'/>

##### 3.1 how to use
```python
from efsm import StateMachine

def update(state, o, s):
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'
 
sm = StateMachine('idle', 'stop')
sm.add('idle', 'move', 'stop', fn=update)  # fn is must in standard mode. data is available.

while sm.step(): ...
print("finish.")
```
```
i'm idle, next to move
i'm moving, next to stop
finish.
```
<span id='32'/>

##### 3.2 You could jump to update other fsm
how to link statemachine:  
```python
from efsm import StateMachine

def update(state, o, s):
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'
 
sm1, sm2 = StateMachine('idle', 'stop'), StateMachine('idle', 'stop')
sm1.add('idle', 'move', 'stop', fn=update)
sm2.add('idle', 'move', 'stop', fn=update)

sm1.link('stop', sm2, 'idle')  # link sm1.stop -> sm2.idle    # when sm1.state come to 'stop', it will start at sm2.idle in next sm1.step

while sm1.step(): ... # note that you only call sm1.step() here
print("finish.")
```
```
i'm idle, next to move
i'm moving, next to stop
i'm idle, next to move
i'm moving, next to stop
finish.
```
But careful for the circle link. It will cause infinite loop.  
<span id='33'/>

##### 3.3 You could create a StateSet before and pass this state_set to add 
```python
from efsm import StateMachine, StateSet

def update(state, o, s):
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'
 
sm = StateMachine('idle', 'stop')
ss = StateSet('idle', 'move', 'stop', fn=update)  # it have the same params as .add
sm.add(ss)  # fn is must in standard mode. data is available.

while sm.step(): ...
print("finish.")
```
```
i'm idle, next to move
i'm moving, next to stop
finish.
```

<span id='34'/>

##### 3.4 By the way that StateSet is not a user object
```python
from efsm import StateSet

ss = StateSet('idle', 'move', 'stop', fn=lambda *a: ...)
print(ss)
print(type(ss))
```
```
[('idle', 'move', 'stop'), <function <lambda> at 0x0000028BC8DD76D0>, <efsm.standard.core.Local object at 0x0000028BC8E269B0>]
<class 'list'>
```
<span id='4'/>

# 4 Shell Mode
It can only deploy on PC python. As the name 'shell' shown, it have all the methods statemachine in standard-mode had, almost like a shell on statemachine in standard-mode. It provide some convienent ways to create fsm.  

该模式只能部署在电脑端的python上。类如其名，这个'shell'就好像标准模式下的StateMachine下加了一个壳。它提供了一些简单方便的途经去生成一个fsm  

<span id='41'/>

##### 4.1 how to use  
Shell-mode use Efsm to replace StateMachine:
```python
from efsm import Efsm, fsm  # efsm offer you a default Efsm instance named fsm

def update(state):  # in shell-mode, you caould pass 0-3 params, it will auto choose how to call it.
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'

# fsm = Efsm()  # 1. use default efsm for showcase  2. Efsm could instance without params
fsm.add('idle', 'move', 'stop', fn=update)

# When efsm has not start, it will try to take the first state added at the first time as start
# so the start is 'idle'

# When efsm has not end, it will try to take the last state added at the first time as end
# so the end is 'stop'

while fsm.step(): ...
print("finish")
```
When we take off all the notes:  
very simplify code, right?
```python
from efsm import Efsm, fsm

def update(state):  # in shell-mode, you caould pass 0-3 params, it will auto choose how to call it.
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'

fsm.add('idle', 'move', 'stop', fn=update)

while fsm.step(): ...
print("finish")
```
```
i'm idle, next to move
i'm moving, next to stop
finish.
```
<span id='42'/>

##### 4.2 but we could use decorater to make it more simple
use decorater to do the same thing:  
```python
from efsm.shell import *

ss = StateSet()  # create a empty StateSet.  # Only use like this way in shell-mode

@ss.idle.move.stop  # only use for function. not the class method
def update(state):  # in shell-mode, you caould pass 0-3 params, it will auto choose how to call it.
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'

fsm.add(update)

while fsm.step(): ...
print("finish")
```
use @fsm before the @ss.idle... could auto add it to fsm
```python
from efsm.shell import *

ss = StateSet()  # create a empty StateSet.  # Only use like this way in shell-mode

@fsm
@ss.idle.move.stop  # only use for function. not the class method
def update(state):    # in shell-mode, you caould pass 0-3 params, it will auto choose how to call it.
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'

while fsm.step(): ...
print("finish")
```
every efsm instance has theirself ss.    
use fsm.ss replace ss could reach the same effect:
```python
from efsm.shell import *

@fsm.ss.idle.move.stop  # only use for function. not the class method
def update(state, o):   # in shell-mode, you caould pass 0-3 params, it will auto choose how to call it.
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'

while fsm.step(): ...
print("finish")
```
```
i'm idle, next to move
i'm moving, next to stop
finish
```
<span id='43'/>

##### 4.3 use doc to do the same thing
you must add @state or @sta or @s to the function's doc:  
```python
from efsm.shell import *

def update(state, o, s):  # in shell-mode, you caould pass 0-3 params, it will auto choose how to call it.
  """
  @state idle move stop  # you could use ',' as the separater.  like idle, move stop
  """
  match state:
      case 'idle':
          print("i'm idle, next to move")
          return "move"
      case 'move':
          print("i'm moving, next to stop")
          return "stop"
  return 'stop'

fsm.add(update)   # this way need call .add

while fsm.step(): ...
print("finish")
```
```
i'm idle, next to move
i'm moving, next to stop
finish
```
<span id='44'/>

##### 4.4 use doc to a class used EfsmMeta 
you could do the similiar thing to a class which use EfsmMeta
```python
from efsm.shell import *

class Test(metaclass=EfsmMeta):  # use EfsmMeta to auto add Test._efsm = Efsm()
  """
  @state idle move stop -> update   # do not same as the function doc, you need to use '->' or ':' ro assign a method in the class to these group states
  """
  
  # __step__ = 'step'  # Auto create a method in this class, used for efsm.step(). Default is 'step', you can overwrite it.
  # def __bool__(self): ...  # Auto create a __bool__ method if you donot define it, Default is efsm.__bool__, you can overwrite it.
  
  def update(self, state, o):  # in shell-mode, you caould pass 0-3 params, it will auto choose how to call it.
    match state:
        case 'idle':
            print("i'm idle, next to move")
            return "move"
        case 'move':
            print("i'm moving, next to stop")
            return "stop"
    return 'stop'

test = Test()

while test.step(): ...  # test doesnot have any other method in efsm.
print("finish")
```
take a compare with the follow example(not use EfsmMeta):
```python
from efsm.shell import *

class Test:
  """
  @state idle move stop -> update  # in this way, the '->' is useless.
  """
  def __call__(self, state, o):  # in shell-mode, you caould pass 0-3 params, it will auto choose how to call it. 
    match state:
        case 'idle':
            print("i'm idle, next to move")
            return "move"
        case 'move':
            print("i'm moving, next to stop")
            return "stop"
    return 'stop'

test = Test()

fsm.add(test)

while fsm.step(): ...
print("finish")
```
  
That's all this python_module-efsm provides.  
Now i will show you few demos in real project:
<span id='5'/>

# 5 demos
<span id='51'/>

## 5.1 Python Demo -Efsm
...
<span id='52'/>

## 5.2 MicroPython Demo -micro or StateMachine
to use in micropython. you need to copy the core.py from efsm/standard/core.py or efsm/micro/core.py to your micropython device, and you can rename it to 'efsm.py'







