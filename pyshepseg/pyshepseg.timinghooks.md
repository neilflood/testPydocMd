# pyshepseg.timinghooks
    A class to support placement of timing points in a Python application.

## Classes
### class AllTests(TestCase)
      Run all tests

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.\_\_init\_\_(self, methodName='runTest')
          Create an instance of the class that will use the named test
          method when executed. Raises a ValueError if the instance does
          not have a method with the specified name.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_multiple(self)

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_nested(self)

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AllTests.test_single(self)

### class Timers
      Manage multiple named timers. See interval() method for example
      usage. The makeSummaryDict() method can be used to generate
      summary statistics on the timings.
      
      Maintains a dictionary of pairs of start/finish times, before and
      after particular operations. These are grouped by operation names,
      and for each name, a list is accumulated of the pairs, for every
      time when this operation was carried out.
      
      The operation names are arbitrary strings chosen by the user at each
      point where a timer is embedded in the application code.
      
      Timing intervals can be safely nested, so some intervals can be
      contained with others.
      
      The object is thread-safe, so multiple threads can accumulate to
      the same names. The object is also pickle-able.
      
      Example Usage::
      
          timings = Timers()
          with timings.interval('walltime'):
              for i in range(count):
                  # Some code with no specific timer
      
                  with timings.interval('reading'):
                      # Code to do reading operations
      
                  with timings.interval('computation'):
                      # Code to do computation
      
          summary = timings.makeSummaryDict()
          print(summary)
      
      The resulting summary dictionary would be something like::
      
          {'walltime': ['total': 12.345, 'min': 1.234, ......],
           'reading': ['total': 3.456, 'min': 0.234, ......],
           'computation': ['total': 9.123, 'min': 1.012, ......]
          }
      
      All times are in seconds.
      
      These 'with interval' blocks can be scattered through an application's
      code, all using the same timings object. The summary dictionary can be
      used to generate a report at the end of the application to present to a
      user, showing how the key parts of the application compare in time taken.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Timers.\_\_init\_\_(self)
          Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Timers.getDurationsForName(self, intervalName)
          For the given interval name, turns that list of start/end times
          into a list of durations (end - start), in seconds.
          
          Returns the list of durations.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Timers.interval(self, intervalName)
          Use as a context manager to time a particular named interval.
          
          Example::
          
              timings = Timers()
              with timings.interval('some_action'):
                  # Code block required to perform the action
          
          After exit from the `with` statement, the timings object will have
          accumulated the start and end times around the code block. These
          will then contribute to the reporting of time intervals.

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Timers.makeSummaryDict(self)
          Make some summary statistics, and return them in a dictionary

#### &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Timers.merge(self, other)
          Merge another Timers object into this one

## Functions
### def mainCmd()

