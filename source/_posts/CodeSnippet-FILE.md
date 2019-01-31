title: CodeSnippet C language
author: Nico
date: 2019-01-31 11:32:45
tags:
---
#### 使用OutputDebugString输出调试信息

```
void __cdecl odprintf(const char* fmt, ...)
{
	char buf[4096], *p = buf;
	va_list args;

	va_start(args, fmt);
	p += vsnprintf_s(p, sizeof(buf), _TRUNCATE, fmt, args);
	va_end(args);

	while (p > buf  &&  isspace(p[-1]))
		*--p = '\0';
	*p++ = '\r';
	*p++ = '\n';
	*p = '\0';

	OutputDebugString(buf);    //OutputDebugString  
}
```
#### 时间戳转字符日期
```
void  __cdecl dateprintf(long long timestamp, char*now,int len) 
{
	struct tm ttime;
	localtime_s(&ttime, &timestamp);
	strftime(now, len, "%Y-%m-%d %H:%M:%S", &ttime);
}
```
#### 按行读取文本文件到map
```
#include<io.h>

std::map<CameraId, int> m_recordloc;

int find_view_by_cid(int cid) {
	auto it = m_recordloc.find(cid);
	return it != m_recordloc.end() ?
		it->second : 1;
}
void  getRecordLocation() {

	odprintf("[HK8600] %s\n", __FUNCTION__);

	if (!_access("device_tr.dat", 0)) {
    
		odprintf("[HK8600] %s\n", "device_tr.dat EXISITS!");
		FILE *file = fopen("device_tr.dat", "r");
		if (file != NULL) {
			while (!feof(file))
			{
				int cameraId, reloc;
				fscanf(file, "%d,%d", &cameraId, &reloc);
				odprintf("[HK8600] cameraId:%d;location:%d\n",cameraId,reloc);
				m_recordloc.insert(std::make_pair(cameraId, reloc));
			}
		}
		else {
			odprintf("[HK8600] %s\n", "Read device_tr.dat FAILED!");
		}
		fclose(file);
	}
	else
		odprintf("[HK8600] %s\n","DOESN'T EXISIT!");

}
```


