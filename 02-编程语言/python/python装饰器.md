# python中的装饰器

### why：为什么需要装饰器

有时候需要在不更改原函数的代码前提下给函数增加新功能

### what：装饰器是什么

>  装饰器本身是一个python函数，它可以在其他函数不需要做任何代码变动的前提下增加额外功能，装饰器的返回值也是一个函数对象。

装饰器就是一个返回函数的高阶函数

### how：怎么写装饰器

#### 装饰器分类

* ##### 简单装饰器

  ```python
  def log(func):
      def warpper(*args, **kw):    # warpper的参数定义决定了warpper函数可以接受任意参数调用
          print 'call %s():' % func.__name__    # 首先打印日志
          return func(*args, **kw)    # 再调用原始函数
      return warpper

  @log
  def now():
      print'haha'

  print now()
  print now.__name__.   # warpper
  ```

  接受一个原函数作为参数，并返回一个新函数

  在返回新函数之前，可以做一些想做的事情

  把装饰器@dec放到函数func定义处的前面，相当于执行了func = dec(func)

* ##### 带参数的装饰器

  如果装饰器本身需要传入参数，则需要编写一个返回decorator的高阶函数，即三层嵌套

  ```python
  def log(text):    # 最外层传入需要的参数
      def decorator(func):    # 第二层传入函数
          #@functools.wraps(func)
          def wrapper(*args, **kw):
              print '%s, %s(): ' % (text, func.__name__)
              return func(*args, **kw)
          return wrapper
      return decorator

  @log('execute')
  def now():
      print 'lala'
      
  print now()
  print now.__name__.   # 结果是warpper
  ```

  因为经过装饰器的函数的`__name__`等属性已经变成了wrapper的，这样会导致有些依赖函数签名的代码出错，所以需要把原始函数的属性复制到wrapper()函数中，这种操作由内置的`functools.wraps`完成

  具体只要在warpper()前面加上`@functools.wraps(func)`即可


* ##### 类装饰器

  只要有某个对象重载了`__call__()`方法，装饰器函数就可以接受这个callable对象作为参数，然后返回一个callable对象

  一般来说callable的都是函数，如果一个类重载了`__call__()`方法，那么这个类对象就有了被调用的行为

  类装饰器就是用`__init__()`接受一个函数，然后重载`__call__()`并返回一个函数

  优点：灵活度大、高内聚、封装性等

  ```python
  class logging(object):
      def __init__(self, func):
          self.func = func
          
      def __call__(self, *args, **kw):
          print 'name: {func}'.format(func = self.func.__name__)
          return self.func(*args, **kw)
      __name__ = 'log'

  @logging
  def now():
      print 'enen'
      
  print now()
  print now.__name__  # log
  ```

* ##### 带参数的类装饰器

  在`__init__()`接受参数并存起来，在`__call__()`中接受函数并返回函数

  ```python
  class logging(object):
      def __init__(self, text):
          self.text = text
      
      def __call__(self, func):    # 接受函数
          @functools.wraps(func)    # 将now的属性复制给wrapper
          def wrapper(*args, **kw):
              print '{text}, {name}'.format(text = self.text, name = func.__name__)
              func(*args, **kw)
          return wrapper
      
  @logging(text='execute')
  def now():
      print 'enen'
      
  print now()
  print now.__name__    # now
  ```

* ##### 内置装饰器

@staticmethod

@classmethod

@property

