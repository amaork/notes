# BMP图像编码
---

BMP图像由3部分组成：

1. BMP文件头 BIT\_FILE\_HEADER
2. BMP信息头 BIT\_INFO\_HEADER
3. BMP图像数据


**文件头数据结构：**
	
	typedef struct {

    	 char type[2];                                /* pictue format for bmp always BM */ 
    	 unsigned long size;                    	 /* whole picure file equal bmp data add bmp file header equal width*height*bpp+file_header_size(54) */
    	 unsigned short reserved1;          /* always zero */
    	 unsigned short reserved2;          /* always zero */
    	 unsigned long offset;                   /* the realy bmp data offset always was file header size 54 */
	} __attribute__((packed))BIT_FILE_HEADER;

 

**信息头数据结构：**

	typedef struct{

    	unsigned long h_size;                    /* bmp information header size just this struct size 40 */
    	long width;                              /* bmp picture width */
    	long height;                             /* bmp picture height */ 
    	unsigned short planes;                   /* always 1 */
    	unsigned short bpp;                      /* bits per pixel */
    	unsigned long compression;           	 /* always 0 */
    	unsigned long i_size;                    /* bmp picture data size */
    	long xppm;                               /* the following elements value both 0 */
    	long yppm;
    	unsigned long coluse;
    	unsigned long clrImportant;

	} BIT_INFO_HEADER;

 

**注意：**

- 文件头数据结构不是四字节对齐的，所以需要使用GNU的\_\_attribute\_\_((packed))语法让数据结构中的数据按照1字节对齐（即保持数据结构原状），否则数据结构的大小将超过14字节，会产生错误。或者使用一次写一个元素的方法把数据结构中的元素写入到BMP文件中去。

- BMP图像数据通常为24位色深，即BIT\_INFO\_HEADER中的bpp的值为24，则用24位来表示一个像素的点，那么RGB三种颜色分别占用8位，在小端内存中其排列：B是第一个在低地址，然后依次是G与R；

- BMP的图形数据还有一个特性，就是其每行数据的长度必须能被4字节整除，如果不满4字节的需要补满4字节，如1366*768的BMP图像，它的每行有1366个点不能被4整除，那么在填写每行的BMP数据的时候需要在每行的末尾添加(4 - (1366%4))个字节用来保证，每行的数据都能被4整除。