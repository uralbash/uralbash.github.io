.. post:: 17 Feb, 2013 17:04
   :tags: python, ThinkPad
   :category: Python
   :author: Uralbash
   :language: ru

ThinkPad управление музыкой с помощью HDAPS
===========================================

| Хорошая статья из Ынтернета как управлять плеером cups наклоняя ноутбук.
| `Ссылка на оригинал <http://www.mhermans.net/dhaps-controlled-thinkpad.html>`_

Сначала установим HDAPS:

.. code-block:: bash

   $ sudo apt-get install tp-smapi-dkms hdapsd

Затем проверим:

.. code-block:: bash

   $ cat /sys/devices/platform/hdaps/position
   (471,504)

И запустим следующий скрипт на :l:`Python`:

.. code-block:: python

   import time, subprocess

   class Orientation(object):
       def __init__(self, readfreq=1):
           self.x_init, self.y_init = self.read()
           self.x_hist = []
           self.y_hist = []

           # read in some values to smooth movement
           self.collect()

           # cooldown makes sure only a single command
           # is executed in one movement
           self.cooldown = 0

       def read(self):
           f = open('/sys/bus/platform/devices/hdaps/position')
           s = f.read().strip()
           f.close()
           x, y = s.strip('()').split(',')

           return int(x), int(y)

       def collect(self, n=10):
           # change n for smoothing over longer period
           while len(self.x_hist) < n+1:
               coords = self.read()
               self.x_hist.append(coords[0])
               self.y_hist.append(coords[1])

       @property
       def x_avg(self):
           return sum(self.x_hist)/len(self.x_hist)

       @property
       def y_avg(self):
           return sum(self.y_hist)/len(self.y_hist)

       def tick(self, timeout=0.1):
           # timeout provides the period between "ticks", i.e.
           # the frequency of reads and checks on thresholds

           self.update() # read in and process new position

           # if not in cooldown period, check if thresholds are
           # triggerd, and possibly execute a command
           if not self.cooldown:
               self.execute()
           else: # reduce cooldown periode every tick
               self.cooldown = self.cooldown - 1

           time.sleep(timeout)

       def update(self):
           self.x, self.y = self.read()

           # position relative to the moving average position
           self.x_rel, self.y_rel = o.x - o.x_avg, o.y - o.y_avg

           # keep a stack of n observations by popping the first
           # and appending to the end
           self.x_hist.pop(0)
           self.x_hist.append(self.x)
           self.y_hist.pop(0)
           self.y_hist.append(self.y)

       def tresholds(self, x_min=-30, x_max=30, y_min=-30, y_max=30):
           # check if thresholds are triggerd, turns a tuple with
           # four True/False values

           triggered = (self.x_rel < x_min, self.x_rel > x_max,
                   self.y_rel < y_min, self.y_rel > y_max)

           return triggered

       def execute(self):
           t = self.tresholds()
           if t == (True, False, False, False):
               print 'Detected a forward tilt'
               subprocess.call(['cmus-remote', '-v', '-10%'])
               self.cooldown = 10

           if t == (False, True, False, False):
               print 'Detected a backward tilt'
               subprocess.call(['cmus-remote', '-v', '+10%'])
               self.cooldown = 10

           if t == (False, False, True, False):
               print 'Detected a left tilt'
               subprocess.call(['cmus-remote', '-u'])
               self.cooldown = 10

           if t == (False, False, False, True):
               print 'Detected a right tilt'
               subprocess.call(['cmus-remote', '-n'])
               self.cooldown = 10

   if __name__ == '__main__':
       o = Orientation()
       while True:
           #print o.x_rel, o.y_rel, o.cooldown
           o.tick()

Все теперь можно запускать ``cups`` и баловаться.
