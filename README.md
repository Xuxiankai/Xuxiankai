#====================================
Error: The size of the connection buffer (131072) was not large enough                                    
to fit a complete line:
  * Increase it by setting `Sys.setenv("VROOM_CONNECTION_SIZE")`
#===================================
Sys.setenv("VROOM_CONNECTION_SIZE" = 655360000)######如果遇到Error: The size of the connection buffer (655360) was not large enough说明VROOM数值太小，需要加大。 

