# rios.riostests.testlayerselection
    Test the input layer selection feature. 
    
    Generates a multi-band image, then applies a RIOS function which only reads 
    two of the layers. Checks that it gets the correct answer for adding them together. 

## Functions
### def doSum(info, inputs, outputs, otherargs)
        Should be given only the two layers we want, therefore can calculate the sum 
        by adding all layers

### def run()
        Run the test

## Values
    TESTNAME = 'TESTLAYERSELECT'
