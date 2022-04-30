# python_module: efsm
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
# 1. General/ 总体描述
efsm is a one-stage state machine. The smallest unit is composed of several **state** and their corresponding **processing function**.

efsm是一个一段式状态机，最小单元是由若干个**状态**和其对应的**处理函数**组成的。  

## State/ 状态
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

## Processing Function/ 处理函数
The processing-function to execute specific tasks and return to the next state according to the current state.  
An available processing function must contain two parameters: state, o  

处理函数的作用是根据当前的状态去执行特定的函数任务以及返回下一个状态。   
一个可用的处理函数必须包含两个参数: state, o  

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

### return:
Jump to the next state by the return value of the processing function. If you return none, it means that there is no state transition  

通过处理函数的返回值跳转到下一个状态。如果返回None，就意味着不进行状态转移.  

```python
def update(state, o):  # proccessing function which only handle: idle, move, stop/ 只负责处理idle, move, stop三种状态的函数
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

## add the smallest unit to fsm
use add(...) to add the smallest unit into your fsm
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


# 2. Micro Mode:
It can deploy on micropython. The simplest fsm in this module only consist of some datum and functions. You could to use this module as a singleton object.  

该模式可以部署在micropython单片机上。这个模块只包含一些函数和数据，你可以把这个module当作一个单例对象



