# bzero, bcopy替代函数
---

当bcopy和bzero函数不能使用的时候，可以使用memmove和memset函数替代：

	#define bzero(b,len)(memset((b),'\0',(len)),(void)0)
	#define bcopy(b1,b2,len)(memmove((b2),(b1),(len)),(void)0)