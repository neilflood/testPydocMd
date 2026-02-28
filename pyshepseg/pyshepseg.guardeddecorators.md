# pyshepseg.guardeddecorators
    A dreadful hack to get around the fact that the numpydoc Sphinx extension
    does not play well with numba's jitclass decorator.
    
    If we are running with Sphinx, then fake the jitclass decorator so that it
    just returns the class it is decorating.

