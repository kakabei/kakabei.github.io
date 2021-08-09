

```c++
int _GetUUID(std::string& strUUID)
{
	boost::uuids::uuid u = boost::uuids::random_generator()();
	if(u.is_nil()) {
		u = boost::uuids::random_generator()();
		if(u.is_nil()) return -1;
	}
	char str[64];
	char *ptr = str;
	for(boost::uuids::uuid::const_iterator i=u.begin(), e=u.end(); i != e; ++i, ptr+=2) 
	{
		snprintf(ptr, 3, "%02x-", *i);
	}
	
	string sTmp = str;
	string s1, s2, s3, s4, s5;
	
	s1 = sTmp.substr(0,8);
	s2 = sTmp.substr(8,4);
	s3 = sTmp.substr(12,4);
	s4 = sTmp.substr(16,4);
	s5 = sTmp.substr(20);
	
	snprintf(str, sizeof(str), "%s-%s-%s-%s-%s", s1.c_str(), s2.c_str(), s3.c_str(), s4.c_str(), s5.c_str());
	
	strUUID = str;
	
	return 0;
}

uint64_t _GetCurrentMS()
{
	struct timeval stCurrentTime;
	gettimeofday(&stCurrentTime, NULL);
	u_int64_t ddwCurrentMilliSeconds = (u_int64_t)stCurrentTime.tv_sec * 1000 + (u_int64_t)stCurrentTime.tv_usec / 1000;
	return ddwCurrentMilliSeconds;
}

time_t _MakeTime(const char *pStr)
{
	struct tm tmLoc;
	char *p;
	char *q;

	while ( !isdigit(*pStr) && *pStr ) ++pStr;
	tmLoc.tm_year = strtol(pStr, &p, 10) - 1900;

	while ( !isdigit(*p) && *p ) ++p;
	tmLoc.tm_mon  = strtol(p, &q, 10) - 1;

	while ( !isdigit(*q) && *q ) ++q;
	tmLoc.tm_mday = strtol(q, &p, 10);

	while ( !isdigit(*p) && *p ) ++p;
	tmLoc.tm_hour = strtol(p, &q, 10);

	while ( !isdigit(*q) && *q ) ++q;
	tmLoc.tm_min  = strtol(q, &p, 10);

	while ( !isdigit(*p) && *p ) ++p;
	tmLoc.tm_sec  = strtol(p, &q, 10);

	return mktime(&tmLoc);
}

// 白名单查找
inline int CheckWhiteList(const std::string& strPin, const std::vector<std::string>& vecWhiteList)
{
	if (!vecWhiteList.empty() && vecWhiteList[0] == "***") return 0;
	if (std::find(vecWhiteList.begin(), vecWhiteList.end(), strPin) != vecWhiteList.end()) return 0;
	return -1;
}
```