# rios.cuiprogress
    Progress bars using CUI interface. 

## Classes
### class CUIProgressBar
      Simple progress as a percentage pronted to the terminal

#### &nbsp;&nbsp;&nbsp;&nbsp; CUIProgressBar.\_\_init\_\_(self)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; CUIProgressBar.displayError(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; CUIProgressBar.displayException(self, trace)

#### &nbsp;&nbsp;&nbsp;&nbsp; CUIProgressBar.displayInfo(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; CUIProgressBar.displayWarning(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; CUIProgressBar.reset(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; CUIProgressBar.setLabelText(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; CUIProgressBar.setProgress(self, progress)

#### &nbsp;&nbsp;&nbsp;&nbsp; CUIProgressBar.setTotalSteps(self, steps)

#### &nbsp;&nbsp;&nbsp;&nbsp; CUIProgressBar.wasCancelled(self)

### class GDALProgressBar
      Uses GDAL's TermProgress to print a progress bar to the terminal

#### &nbsp;&nbsp;&nbsp;&nbsp; GDALProgressBar.\_\_init\_\_(self)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; GDALProgressBar.displayError(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; GDALProgressBar.displayException(self, trace)

#### &nbsp;&nbsp;&nbsp;&nbsp; GDALProgressBar.displayInfo(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; GDALProgressBar.displayWarning(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; GDALProgressBar.reset(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; GDALProgressBar.setLabelText(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; GDALProgressBar.setProgress(self, progress)

#### &nbsp;&nbsp;&nbsp;&nbsp; GDALProgressBar.setTotalSteps(self, steps)

#### &nbsp;&nbsp;&nbsp;&nbsp; GDALProgressBar.wasCancelled(self)

### class LogProgressBar
      A progress object specifically for printing to logs.
      The other progress objects here don't print to logs - 
      only terminals. This saves unecessary info being put
      into the log but does make it difficult to monitor a 
      job from the logs.
      To avoid too much information being saved to the log
      only unique percentages are printed.

#### &nbsp;&nbsp;&nbsp;&nbsp; LogProgressBar.\_\_init\_\_(self)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; LogProgressBar.displayError(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; LogProgressBar.displayException(self, trace)

#### &nbsp;&nbsp;&nbsp;&nbsp; LogProgressBar.displayInfo(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; LogProgressBar.displayWarning(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; LogProgressBar.reset(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; LogProgressBar.setLabelText(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; LogProgressBar.setProgress(self, progress)

#### &nbsp;&nbsp;&nbsp;&nbsp; LogProgressBar.setTotalSteps(self, steps)

#### &nbsp;&nbsp;&nbsp;&nbsp; LogProgressBar.wasCancelled(self)

### class SilentProgress
      A progress object which is completely silent. 

#### &nbsp;&nbsp;&nbsp;&nbsp; SilentProgress.\_\_init\_\_(self)
        Initialize self.  See help(type(self)) for accurate signature.

#### &nbsp;&nbsp;&nbsp;&nbsp; SilentProgress.displayError(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; SilentProgress.displayException(self, trace)

#### &nbsp;&nbsp;&nbsp;&nbsp; SilentProgress.displayInfo(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; SilentProgress.displayWarning(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; SilentProgress.reset(self)

#### &nbsp;&nbsp;&nbsp;&nbsp; SilentProgress.setLabelText(self, text)

#### &nbsp;&nbsp;&nbsp;&nbsp; SilentProgress.setProgress(self, progress)

#### &nbsp;&nbsp;&nbsp;&nbsp; SilentProgress.setTotalSteps(self, steps)

#### &nbsp;&nbsp;&nbsp;&nbsp; SilentProgress.wasCancelled(self)

