# Implementation settings for (very) simple Vivado projects

https://support.xilinx.com/s/article/58992?language=en_US

> [Place 30-415] IO Placement failed due to over utilization

Project Manager => Settings => Synthesis => More options. Add this flag:
```
-mode out_of_context
```




