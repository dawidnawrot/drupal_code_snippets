If you dump some variable by ksm(), it's data is going to be divided into three section:

 - Contents
 - Available methods
 - Static class properties
 
 If you want to use any of contents you can access public contents by calling
 
 ```
 $variables->some_contents
 ```
 
 Here is the example:
 
