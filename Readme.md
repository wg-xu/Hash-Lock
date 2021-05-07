@[TOC](Hashlock)
# 一、原理
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210507125155393.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZvdXJpZXJGaXNoZXI=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210507125104106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZvdXJpZXJGaXNoZXI=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210507125131652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZvdXJpZXJGaXNoZXI=,size_16,color_FFFFFF,t_70)

# 二、实现结果
## 1、验证成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210507180035485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZvdXJpZXJGaXNoZXI=,size_16,color_FFFFFF,t_70)
## 2、验证失败
### 不在数据库
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210507180126788.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZvdXJpZXJGaXNoZXI=,size_16,color_FFFFFF,t_70)
### 无效密钥Key
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210507180248551.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZvdXJpZXJGaXNoZXI=,size_16,color_FFFFFF,t_70)

# 三、源代码

```c
// Hash-Lock.c
/********************************************************************************* 
  *Copyright(C),Fisher1016 
  *FileName:  Hash-Lock.c
  *Author:  xwang
  *Date:  2020.5.7 
  *Description: RFID electronic tag verification is realized through hash function
**********************************************************************************/  
#include <stdio.h>
#include <math.h>
#include <stdlib.h>
#define Tag_N 3
#define Data_N 10
#define Hash_core 17    //hash算法的核心机密
struct ElecTag
{
	int metalID;
	int ID;
};

struct DataBase
{
	int metalID;
	int Key;
	int ID;
};

ElecTag Tag[Tag_N];
DataBase Data[Data_N];
void Init() {
	int i;
	for (i = 0; i < Tag_N; i++) {
		Tag[i].metalID = rand();
		Tag[i].ID = rand();
	}
	for (i = 0; i < Data_N; i++) {
		Data[i].metalID = rand();
		Data[i].Key = rand();
		Data[i].ID = rand();
	}
}
// Administrator define Tag Values and modifies database
void AdminiWrite() {
	Tag[0].metalID = 14;  //it not exist in DataBase
	Tag[0].ID = 66666666;
	Tag[1].metalID = 15;
	Tag[1].ID = 66666666;
	Tag[2].metalID = 16;
	Tag[2].ID = 66666666;

	Data[0].metalID = 15;
	Data[0].Key = 32;
	Data[0].ID = 66666666;

	Data[1].metalID = 16;
	Data[1].Key = 32;  //true value is 33 ( 33 mod 17 =16 )
	Data[1].ID = 66666666;
}

//检测标签
int Query(ElecTag* Tag) {
	ElecTag* pTag = Tag;
	printf("\n**************************************************\n");
	printf("The Tag's metalID has been read in\n");
	return pTag->metalID;
}

//by Reader,return Data in Database to Reader
DataBase GetData(int RMetalID) {
	int i;
	for (i = 0; i < Data_N; i++) {
		if (RMetalID == Data[i].metalID) {
			printf("\n**************************************************\n");
			printf("Find this Tag in DataBase!\n");
			return Data[i];
		}
	}
	printf("\n**************************************************\n");
	printf("Error:Can not find this Tag in DataBase!\n");
	exit(-1);
}


int Hash(int Key) {
	return Key % Hash_core;
}
int GetTagID(DataBase * RevData, ElecTag * Tag) {
	DataBase* pData = RevData;
	ElecTag* pTag = Tag;
	int i;
	if (Hash(pData->Key) == pTag->metalID) {
		printf("\n**************************************************\n");
		printf("Electronic tag verification successful!\n");
		return pTag->ID;
	}
	printf("\n**************************************************\n");
	printf("Error:Electronic tag verification failed!\n");
	exit(-1);
}


void ReaderVer(int RevDataID, int TagID) {
	if (RevDataID == TagID) {
		printf("\n**************************************************\n");
		printf("Vertify Successfully!\n");
		printf("The ID is %d",TagID);
	}
	else {
		printf("\n**************************************************\n");
		printf("Error:ID does not match!\n");
	}
}
//Reader is core
int main() {
	Init();
	AdminiWrite();
	//you can choose "Test_Tag" equel to 0 or 2,but I set they are unabled to read
	int Test_Tag = 1;
	int RMetalID = Query(&Tag[Test_Tag]);
	DataBase RevData = GetData(RMetalID);
	int TagID = GetTagID(&RevData, &Tag[Test_Tag]);
	ReaderVer(RevData.ID, TagID);
}
```

