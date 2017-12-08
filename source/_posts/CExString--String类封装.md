title: CExString --String类封装
author: ccbu
tags: []
categories:
  - cpp
date: 2017-12-05 22:01:00
---

ExString.h
----
```
#define MIN_CAPACITY         15
#define MAX_INIT_LENGTH      1024*4

class CExString
{
public:
	CExString();
	CExString(const int nLen);
	CExString(const CExString& src);
	CExString(const LPCSTR lpsz, int nLen = -1);
	~CExString();

	void Empty();
	int GetLength() const;
	int GetCapacity() const;
	//bool IsEmpty() const;
	CHAR GetAt(int nIndex) const;
	LPCSTR GetData() const;
	void SetAt(int nIndex, CHAR ch);
	//CExString Trim() const;
	
	
	operator LPCSTR() const;
	CHAR operator[] (int nIndex) const;
	const CExString& operator=(const CExString& src);
	const CExString& operator=(LPCSTR pstr);
#ifdef _UNICODE
	const CExString& operator=(LPCWSTR lpwStr);
	const CExString& operator+=(LPCWSTR lpwStr);
#endif	
	CExString operator+(const CExString& src) const;
	CExString operator+(LPCSTR pstr) const;
	const CExString& operator+=(const CExString& src);
	const CExString& operator+=(LPCSTR lpStr);

	bool operator == (LPCSTR lpStr) const;
	bool operator != (LPCSTR lpStr) const;
	bool operator <= (LPCSTR lpStr) const;
	bool operator <  (LPCSTR lpStr) const;
	bool operator >= (LPCSTR lpStr) const;
	bool operator >  (LPCSTR lpStr) const;

	int Compare(LPCSTR lpStr) const;
	int CompareNoCase(LPCSTR lpStr) const;

	void MakeUpper();
	void MakeLower();

	CExString Left(int iLength) const;
	CExString Mid(int iPos, int iLength = -1) const;
	CExString Right(int iLength) const;

	int Find(CHAR ch, int iPos = 0) const;
	int Find(LPCSTR pstr, int iPos = 0) const;
	int ReverseFind(CHAR ch) const;
	int Replace(LPCSTR pstrFrom, LPCSTR pstrTo);

	int __cdecl Format(LPCSTR pstrFormat, ...);
	int __cdecl SmallFormat(LPCSTR pstrFormat, ...);

private:
	void SetLength(int iLength);
	int ReallocData(int iCount);

	void Append(LPCSTR pwstr, int nLength = -1);
	void Assign(LPCSTR pwstr, int nLength = -1);

	

protected:
	int m_iCapacity;
	int m_iLength;
	LPSTR m_pstr;
};

#ifdef _UNICODE
class CExwString
{
public:
	CExwString(void);
	CExwString(const int nLen);
	CExwString(const CExwString& src);
	CExwString(const LPCWSTR lpwsz, int nLen = -1);
	~CExwString(void);

	void Empty();
	int GetLength() const;
	int GetCapacity() const;
	//bool IsEmpty() const;
	WCHAR GetAt(int nIndex) const;
	LPCWSTR GetData() const;
	void SetAt(int nIndex, WCHAR ch);



	operator LPCWSTR() const;
	WCHAR operator[] (int nIndex) const;
	const CExwString& operator=(const CExwString& src);
	const CExwString& operator=(LPCWSTR lpwStr);

	const CExwString& operator=(LPCSTR lpStr);
	const CExwString& operator+=(LPCSTR lpStr);
	
	CExwString operator+(const CExwString& src) const;
	CExwString operator+(LPCWSTR lpwStr) const;
	const CExwString& operator+=(const CExwString& src);
	const CExwString& operator+=(LPCWSTR lpwStr);

	bool operator == (LPCWSTR lpwStr) const;
	bool operator != (LPCWSTR lpwStr) const;
	bool operator <= (LPCWSTR lpwStr) const;
	bool operator <  (LPCWSTR lpwStr) const;
	bool operator >= (LPCWSTR lpwStr) const;
	bool operator >  (LPCWSTR lpwStr) const;

	int Compare(LPCWSTR lpwStr) const;
	int CompareNoCase(LPCWSTR lpwStr) const;

	void MakeUpper();
	void MakeLower();

	CExwString Left(int iLength) const;
	CExwString Mid(int iPos, int iLength = -1) const;
	CExwString Right(int iLength) const;

	int Find(WCHAR ch, int iPos = 0) const;
	int Find(LPCWSTR pwstr, int iPos = 0) const;
	int ReverseFind(WCHAR ch) const;
	int Replace(LPCWSTR pwstrFrom, LPCWSTR pwstrTo);

	int __cdecl Format(LPCWSTR pwstrFormat, ...);
	int __cdecl SmallFormat(LPCWSTR pwstrFormat, ...);

private:
	void SetLength(int iLength);
	int ReallocData(int iCount);

	void Append(LPCWSTR pstr, int nLength = -1);
	void Assign(LPCWSTR pstr, int nLength = -1);

protected:
	int m_iCapacity;
	int m_iLength;
	LPWSTR m_pwstr;

};
#endif
```
ExString.cpp
----
```
#include "ExString.h"


CExString::CExString():m_iLength(0),m_iCapacity(0),m_pstr(NULL)
{

}

CExString::CExString( const int nLen ):m_iLength(0),m_iCapacity(0),m_pstr(NULL)
{
	int iLength = nLen;
	if(iLength > 0)
	{
		if(iLength > MAX_INIT_LENGTH) iLength = MAX_INIT_LENGTH;
		m_iCapacity = ReallocData(iLength);
	}
}


CExString::CExString( const CExString& src ):m_iLength(0),m_iCapacity(0),m_pstr(NULL)
{
	if(src.GetLength()>0)
	{
		Assign(src.GetData(), src.GetLength());
	}
}

CExString::CExString( const LPCSTR lpsz, int nLen /*= -1*/ ):m_iLength(0),m_iCapacity(0),m_pstr(NULL)
{
	if(lpsz == NULL || nLen == 0) return;
	Assign(lpsz, nLen);
}

CExString::~CExString()
{
	Empty();
}

void CExString::Empty()
{
	m_iCapacity = 0;
	m_iLength = 0;
	if(m_pstr != NULL)
	{
		free(m_pstr);
		m_pstr = NULL;
	}
}

int CExString::GetLength() const
{
	return m_iLength;
}

int CExString::GetCapacity() const
{
	return m_iCapacity;
}

//bool CExString::IsEmpty() const
//{
//	if(m_pstr == NULL) return true;
//	return m_pstr[0] == '\0';
//}

int CExString::ReallocData( int iCount )
{
	if(m_pstr == NULL)
	{
		int capacity = (iCount | MIN_CAPACITY) + 1;
		m_pstr = (CHAR*)malloc(capacity);
		if(m_pstr != NULL)
		{
			m_pstr[capacity-1] = '\0';		
			return capacity;
		}
	}
	else
	{
		int capacity = (iCount | MIN_CAPACITY) + 1;
		CHAR* pNewStr = (CHAR*)(realloc((LPVOID)m_pstr,capacity));
		if(pNewStr != NULL)
		{
			m_pstr = pNewStr;
			m_pstr[capacity-1] = '\0';
			return capacity;
		}
	}
	
	return 0;
}

void CExString::Append( LPCSTR pstr, int nLength /*= -1*/)
{
	if(pstr == NULL) return;

	int iLength = (nLength < 0) ? strlen(pstr) : nLength;
	if(iLength < 1) return;

	int newLength = GetLength() + iLength;
	
	if(newLength >= m_iCapacity)
	{
		m_iCapacity = ReallocData(newLength);
	}

	memmove(m_pstr+m_iLength,pstr,iLength);
	SetLength(newLength);
	m_pstr[newLength] = '\0';
}

void CExString::Assign( LPCSTR pstr, int nLength /*= -1*/ )
{
	if(pstr == NULL) return;

	int iLength = (nLength == -1) ? strlen(pstr) : nLength;
	if(iLength < 1) return;
	
	if(iLength >= m_iCapacity)
	{
		m_iCapacity = ReallocData(iLength);
	}

	memcpy(m_pstr, pstr, iLength);
	SetLength(iLength);
	m_pstr[iLength] = '\0';
}

LPCSTR CExString::GetData() const
{
	return m_pstr;
}

CHAR CExString::GetAt( int nIndex ) const
{
	ASSERT( m_pstr != NULL && (nIndex>= 0 && nIndex < m_iLength));
	return m_pstr[nIndex];
}

void CExString::SetAt( int nIndex, CHAR ch )
{
	ASSERT( m_pstr != NULL && (nIndex>= 0 && nIndex < m_iLength));
	m_pstr[nIndex] = ch;
}

CExString::operator LPCSTR() const
{
	return m_pstr;
}

CHAR CExString::operator[]( int nIndex ) const
{
	return GetAt(nIndex);
}

const CExString& CExString::operator=( const CExString& src )
{
	if(src.GetLength() == 0)
	{
		Empty();
	}
	else
	{
		Assign(src.GetData(),src.GetLength());
	}
	
	return *this;
}

const CExString& CExString::operator=( LPCSTR pstr )
{
	if(pstr)
	{
		ASSERT(!::IsBadStringPtrA(pstr,-1));
		Assign(pstr);
	}
	else
	{
		Empty();
	}
	
	return *this;
}

#ifdef _UNICODE

const CExString& CExString::operator=( LPCWSTR lpwStr )
{
	if ( lpwStr )
	{
		ASSERT(!::IsBadStringPtrW(lpwStr,-1));

		int iwLenght = ((int) wcslen(lpwStr) * 2) + 1;
		LPSTR pstr = (LPSTR) _alloca(iwLenght);
		pstr[iwLenght-1] = '\0';
		if( pstr != NULL ) ::WideCharToMultiByte(::GetACP(), 0, lpwStr, -1, pstr, iwLenght, NULL, NULL);
		Assign(pstr);
	}
	else
	{
		Empty();
	}

	return *this;
}

const CExString& CExString::operator+=( LPCWSTR lpwStr )
{
	if ( lpwStr )
	{
		ASSERT(!::IsBadStringPtrW(lpwStr,-1));
		int iwLenght = ((int) wcslen(lpwStr) * 2) + 1;
		LPSTR pstr = (LPSTR) _alloca(iwLenght);
		pstr[iwLenght-1] = '\0';
		if( pstr != NULL ) ::WideCharToMultiByte(::GetACP(), 0, lpwStr, -1, pstr, iwLenght, NULL, NULL);
		Append(pstr);
	}

	return *this;
}

#endif

const CExString& CExString::operator+=( const CExString& src )
{
	if(src.GetLength()>0)
	{
		Append(src.GetData(),src.GetLength());
	}

	return *this;
}

const CExString& CExString::operator+=( LPCSTR lpStr )
{
	if(lpStr)
	{
		ASSERT(!::IsBadStringPtrA(lpStr,-1));
		Append(lpStr);
	}

	return *this;
}

CExString CExString::operator+( const CExString& src ) const
{
	if(src.GetLength()>0)
	{
		CExString tmpStr = *this;
		tmpStr.Append(src);
		return tmpStr;
	}
	
	return *this;
}

CExString CExString::operator+( LPCSTR lpStr ) const
{
	if(lpStr)
	{
		ASSERT(!::IsBadStringPtrA(lpStr,-1));
		CExString tmpStr = *this;
		tmpStr.Append(lpStr);
		return tmpStr;
	}
	return *this;
}

bool CExString::operator==( LPCSTR lpStr ) const
{
	return (Compare(lpStr) == 0);
}

bool CExString::operator!=( LPCSTR lpStr ) const
{
	return (Compare(lpStr) != 0);
}

bool CExString::operator<=( LPCSTR lpStr ) const
{
	return (Compare(lpStr) <= 0);
}

bool CExString::operator<( LPCSTR lpStr ) const
{
	return (Compare(lpStr) < 0);
}

bool CExString::operator>=( LPCSTR lpStr ) const
{
	return (Compare(lpStr) >= 0);
}

bool CExString::operator>( LPCSTR lpStr ) const
{
	return (Compare(lpStr) > 0);
}

int CExString::Compare( LPCSTR lpStr ) const
{
	ASSERT(lpStr != NULL);
	if(!lpStr) return 1;
	if(m_pstr == NULL) return -1;
	return strcmp(m_pstr, lpStr); 
}

int CExString::CompareNoCase( LPCSTR lpStr ) const
{
	ASSERT(lpStr != NULL);
	if(!lpStr) return 1;
	if(m_pstr == NULL) return -1;
	return _stricmp(m_pstr, lpStr); 
}

void CExString::MakeUpper()
{
	if(m_pstr != NULL)
	{
		_strupr_s(m_pstr, m_iLength+1);//长度中需包含'\0'
	}
}

void CExString::MakeLower()
{
	if(m_pstr != NULL)
	{
		_strlwr_s(m_pstr, m_iLength+1);
	}
}

CExString CExString::Left( int iLength ) const
{
	if( iLength < 0 ) iLength = 0;
	if( iLength > GetLength() ) iLength = GetLength();
	return CExString(m_pstr, iLength);
}

CExString CExString::Mid( int iPos, int iLength /*= -1*/ ) const
{
	if( iPos < 0 ) iPos = 0;
	if( iLength < 0 ) iLength = GetLength() - iPos;
	if( iPos + iLength > GetLength() ) iLength = GetLength() - iPos;
	if( iLength <= 0 ) return CExString();
	return CExString(m_pstr + iPos, iLength);
}

CExString CExString::Right( int iLength ) const
{
	int Length = (iLength < 0) ? 0 : iLength;
	int iPos = GetLength() - Length;
	if( iPos < 0) 
	{
		iPos = 0;
		Length = GetLength();
	}
	return CExString(m_pstr + iPos, Length);
}

int CExString::Find( CHAR ch, int iPos /*= 0*/ ) const
{
	if( iPos < 0 || iPos >= GetLength() || m_pstr == NULL ) return -1;
	LPCSTR p = strchr(m_pstr + iPos, ch);
	if( p == NULL ) return -1;
	return (int)(p - m_pstr);
}

int CExString::Find( LPCSTR pstr, int iPos /*= 0*/ ) const
{
	if( iPos < 0 || iPos >= GetLength() || m_pstr == NULL ) return -1;
	LPCSTR p = strstr(m_pstr + iPos, pstr);
	if( p == NULL ) return -1;
	return (int)(p - m_pstr);
}

int CExString::ReverseFind( CHAR ch ) const
{
	if(m_pstr == NULL) return -1;
	LPCSTR p = strrchr(m_pstr, ch);
	if( p == NULL ) return -1;
	return (int)(p - m_pstr);
}

int CExString::Replace( LPCSTR pstrFrom, LPCSTR pstrTo )
{
	int nCount = 0;
	int iStart = 0;
	int iPos = Find(pstrFrom);
	if( iPos < 0 ) return 0;
	int cchFrom = (int) strlen(pstrFrom);
	int cchTo = (int) strlen(pstrTo);
	CExString sTemp(GetCapacity());
	while( iPos >= 0 ) {
		sTemp.Append(m_pstr+iStart,iPos-iStart);
		sTemp += pstrTo;
		iStart = iPos+cchFrom;
		iPos = Find(pstrFrom, iPos + cchFrom);
		nCount++;
	}
	sTemp.Append(m_pstr+iStart,GetLength()-iStart);
	Assign(sTemp);
	return nCount;
}

int __cdecl CExString::Format( LPCSTR pstrFormat, ... )
{
	CHAR szBuffer[1024] = { 0 };
	va_list argList;
	va_start(argList, pstrFormat);
	int iRet = ::wvsprintfA(szBuffer, pstrFormat, argList);
	va_end(argList);
	Assign(szBuffer);
	return iRet;
}

int __cdecl CExString::SmallFormat( LPCSTR pstrFormat, ... )
{
	CHAR szBuffer[64] = { 0 };
	va_list argList;
	va_start(argList, pstrFormat);
	int iRet = ::wvsprintfA(szBuffer,pstrFormat, argList);
	va_end(argList);
	Assign(szBuffer);
	return iRet;
}

void CExString::SetLength( int iLength )
{
	m_iLength = iLength;
}

//CExString CExString::Trim() const
//{
//	return CExString();
//
//}


#ifdef _UNICODE

CExwString::CExwString( void ) : m_iLength(0),m_iCapacity(0),m_pwstr(NULL)
{

}

CExwString::CExwString( const int nLen ) : m_iLength(0),m_iCapacity(0),m_pwstr(NULL)
{
	int iLength = nLen;
	if(iLength > 0)
	{
		if(iLength > MAX_INIT_LENGTH) iLength = MAX_INIT_LENGTH;
		m_iCapacity = ReallocData(iLength);
	}
}

CExwString::CExwString( const CExwString& src ) : m_iLength(0),m_iCapacity(0),m_pwstr(NULL)
{
	if(src.GetLength() > 0)
	{
		Assign(src.GetData(), src.GetLength());
	}
}

CExwString::CExwString( const LPCWSTR lpwsz, int nLen /*= -1*/ ) : m_iLength(0),m_iCapacity(0),m_pwstr(NULL)
{
	if(lpwsz == NULL || nLen == 0) return;
	Assign(lpwsz, nLen);
}

CExwString::~CExwString( void )
{
	Empty();
}

void CExwString::Empty()
{
	m_iLength = 0;
	m_iCapacity = 0;
	if(m_pwstr != NULL)
	{
		free(m_pwstr);
		m_pwstr = NULL;
	}
}

int CExwString::GetLength() const
{
	return m_iLength;
}

int CExwString::GetCapacity() const
{
	return m_iCapacity;
}

//bool CExwString::IsEmpty() const
//{
//	if(m_pwstr == NULL) return true;
//	return m_pwstr[0] == L'\0';
//}

WCHAR CExwString::GetAt( int nIndex ) const
{
	ASSERT(m_pwstr != NULL && (nIndex >= 0 && nIndex < m_iLength));
	return m_pwstr[nIndex];
}

LPCWSTR CExwString::GetData() const
{
	return m_pwstr;
}

void CExwString::SetAt( int nIndex, WCHAR ch )
{
	ASSERT(m_pwstr != NULL && (nIndex >= 0 && nIndex < m_iLength));
	m_pwstr[nIndex] = ch;
}

CExwString::operator LPCWSTR() const
{
	return m_pwstr;
}

WCHAR CExwString::operator[]( int nIndex ) const
{
	return GetAt(nIndex);
}

const CExwString& CExwString::operator=( const CExwString& src )
{
	if(src.GetLength() == 0)
	{
		Empty();
	}
	else
	{
		Assign(src.GetData(),src.GetLength());
	}

	return *this;
}

const CExwString& CExwString::operator=( LPCWSTR lpwStr )
{
	if(lpwStr)
	{
		ASSERT(!::IsBadStringPtrW(lpwStr, -1));
		Assign(lpwStr);
	}
	else
	{
		Empty();
	}

	return *this;
}

const CExwString& CExwString::operator=( LPCSTR lpStr )
{
	if(lpStr)
	{
		ASSERT(!::IsBadStringPtrA(lpStr, -1));

		int iLenght = ((int)strlen(lpStr) + 1);
		LPWSTR pwstr = (LPWSTR) _alloca(iLenght*sizeof(WCHAR));
		pwstr[iLenght-1] = '\0';
		if( pwstr != NULL ) 
		{
			::MultiByteToWideChar(::GetACP(), 0, lpStr, -1, pwstr, iLenght);
		}
		Assign(pwstr);
	}
	else
	{
		Empty();
	}

	return *this;
}

const CExwString& CExwString::operator+=( LPCSTR lpStr )
{
	if(lpStr)
	{
		ASSERT(!::IsBadStringPtrA(lpStr, -1));

		int iLenght = ((int)strlen(lpStr) + 1);
		LPWSTR pwstr = (LPWSTR) _alloca(iLenght*sizeof(WCHAR));
		pwstr[iLenght-1] = '\0';
		if( pwstr != NULL ) 
		{
			::MultiByteToWideChar(::GetACP(), 0, lpStr, -1, pwstr, iLenght);
		}
		Append(pwstr);
	}

	return *this;
}

const CExwString& CExwString::operator+=( const CExwString& src )
{
	if(src.GetLength() > 0)
	{
		Append(src.GetData(),src.GetLength());
	}

	return *this;
}

const CExwString& CExwString::operator+=( LPCWSTR lpwStr )
{
	if(lpwStr)
	{
		ASSERT(!::IsBadStringPtrW(lpwStr, -1));
		Append(lpwStr);
	}

	return *this;
}


CExwString CExwString::operator+( const CExwString& src ) const
{
	if(src.GetLength() > 0)
	{
		CExwString tmpwStr = *this;
		tmpwStr.Append(src.GetData(), src.GetLength());
		return tmpwStr;
	}
	return *this;
}

CExwString CExwString::operator+( LPCWSTR lpwStr ) const
{
	if(lpwStr)
	{
		ASSERT(!::IsBadStringPtrW(lpwStr, -1));
		CExwString tmpwStr = *this;
		tmpwStr.Append(lpwStr);
		return tmpwStr;
	}

	return *this;
}

bool CExwString::operator==( LPCWSTR lpwStr ) const
{
	return (Compare(lpwStr) == 0);
}

bool CExwString::operator!=( LPCWSTR lpwStr ) const
{
	return (Compare(lpwStr) != 0);
}

bool CExwString::operator<=( LPCWSTR lpwStr ) const
{
	return (Compare(lpwStr) <= 0);
}

bool CExwString::operator<( LPCWSTR lpwStr ) const
{
	return (Compare(lpwStr) < 0);
}

bool CExwString::operator>=( LPCWSTR lpwStr ) const
{
	return (Compare(lpwStr) >= 0);
}

bool CExwString::operator>( LPCWSTR lpwStr ) const
{
	return (Compare(lpwStr) > 0);
}

int CExwString::Compare( LPCWSTR lpwStr ) const
{
	ASSERT(lpwStr != NULL);
	if(!lpwStr) return 1;
	if(!m_pwstr) return -1;
	return wcscmp(m_pwstr, lpwStr);
}

int CExwString::CompareNoCase( LPCWSTR lpwStr ) const
{
	ASSERT(lpwStr != NULL);
	if(!lpwStr) return 1;
	if(!m_pwstr) return -1;
	return _wcsicmp(m_pwstr, lpwStr);
}

void CExwString::MakeUpper()
{
	if(m_pwstr != NULL)
	{
		_wcsupr_s(m_pwstr, m_iLength+1);//长度包括'\0'
	}
}

void CExwString::MakeLower()
{
	if(m_pwstr != NULL)
	{
		_wcslwr_s(m_pwstr, m_iLength+1);
	}
}

CExwString CExwString::Left( int iLength ) const
{
	if( iLength < 0 ) iLength = 0;
	if( iLength > GetLength() ) iLength = GetLength();
	return CExwString(m_pwstr, iLength);
}

CExwString CExwString::Mid( int iPos, int iLength /*= -1*/ ) const
{
	if( iPos < 0 ) iPos = 0;
	if( iLength < 0 ) iLength = GetLength() - iPos;
	if( iPos + iLength > GetLength() ) iLength = GetLength() - iPos;
	if( iLength <= 0 ) return CExwString();
	return CExwString(m_pwstr + iPos, iLength);
}

CExwString CExwString::Right( int iLength ) const
{
	int Length = (iLength < 0) ? 0 : iLength;
	int iPos = GetLength() - Length;
	if( iPos < 0) 
	{
		iPos = 0;
		Length = GetLength();
	}
	return CExwString(m_pwstr + iPos, Length);
}

int CExwString::Find( WCHAR ch, int iPos /*= 0*/ ) const
{
	if( iPos < 0 || iPos >= GetLength() || m_pwstr == NULL ) return -1;
	LPCWSTR p = wcschr(m_pwstr + iPos, ch);
	if( p == NULL ) return -1;
	return (int)(p - m_pwstr);
}

int CExwString::Find( LPCWSTR pwstr, int iPos /*= 0*/ ) const
{
	if( iPos < 0 || iPos >= GetLength() || m_pwstr == NULL ) return -1;
	LPCWSTR p = wcsstr(m_pwstr + iPos, pwstr);
	if( p == NULL ) return -1;
	return (int)(p - m_pwstr);
}

int CExwString::ReverseFind( WCHAR ch ) const
{
	if(m_pwstr == NULL) return -1;
	LPCWSTR p = wcsrchr(m_pwstr, ch);
	if( p == NULL ) return -1;
	return (int)(p - m_pwstr);
}

int CExwString::Replace( LPCWSTR pwstrFrom, LPCWSTR pwstrTo )
{
	int nCount = 0;
	int iStart = 0;
	int iPos = Find(pwstrFrom);
	if( iPos < 0 ) return 0;
	int cchFrom = (int) wcslen(pwstrFrom);
	int cchTo = (int) wcslen(pwstrTo);
	CExwString sTemp(GetCapacity());
	while( iPos >= 0 ) {
		sTemp.Append(m_pwstr+iStart,iPos-iStart);
		sTemp += pwstrTo;
		iStart = iPos+cchFrom;
		iPos = Find(pwstrFrom, iPos + cchFrom);
		nCount++;
	}
	sTemp.Append(m_pwstr+iStart,GetLength()-iStart);
	Assign(sTemp);
	return nCount;
}

int __cdecl CExwString::Format( LPCWSTR pwstrFormat, ... )
{
	WCHAR szBuffer[1024] = { 0 };
	va_list argList;
	va_start(argList, pwstrFormat);
	int iRet = ::wvsprintfW(szBuffer, pwstrFormat, argList);
	va_end(argList);
	Assign(szBuffer);
	return iRet;
}

int __cdecl CExwString::SmallFormat( LPCWSTR pwstrFormat, ... )
{
	WCHAR szBuffer[64] = { 0 };
	va_list argList;
	va_start(argList, pwstrFormat);
	int iRet = ::wvsprintfW(szBuffer,pwstrFormat, argList);
	va_end(argList);
	Assign(szBuffer);
	return iRet;
}

void CExwString::SetLength( int iLength )
{
	m_iLength = iLength;
}

int CExwString::ReallocData( int iCount )
{
	if(m_pwstr == NULL)
	{
		int capacity = (iCount | MIN_CAPACITY) + 1;
		m_pwstr = (WCHAR*)malloc(capacity*sizeof(WCHAR));
		if(m_pwstr != NULL)
		{
			m_pwstr[capacity-1] = '\0';		
			return capacity;
		}
	}
	else
	{
		int capacity = (iCount | MIN_CAPACITY) + 1;
		WCHAR* pNewStr = (WCHAR*)(realloc((LPVOID)m_pwstr,capacity*sizeof(WCHAR)));
		if(pNewStr != NULL)
		{
			m_pwstr = pNewStr;
			m_pwstr[capacity-1] = '\0';
			return capacity;
		}
	}

	return 0;
}

void CExwString::Append( LPCWSTR pwstr, int nLength /*= -1*/ )
{
	if(pwstr == NULL) return;

	int iLength = (nLength < 0) ? wcslen(pwstr) : nLength;
	if(iLength < 1) return;

	int newLength = GetLength() + iLength;

	if(newLength >= m_iCapacity)
	{
		m_iCapacity = ReallocData(newLength);
	}

	memmove(m_pwstr + m_iLength, pwstr, iLength*sizeof(WCHAR));
	SetLength(newLength);
	m_pwstr[newLength] = '\0';
}

void CExwString::Assign( LPCWSTR pwstr, int nLength /*= -1*/ )
{
	if(pwstr == NULL) return;

	int iLength = (nLength == -1) ? wcslen(pwstr) : nLength;
	if(iLength < 1) return;

	if(iLength >= m_iCapacity)
	{
		m_iCapacity = ReallocData(iLength);
	}

	memcpy(m_pwstr, pwstr, iLength*sizeof(WCHAR));
	SetLength(iLength);
	m_pwstr[iLength] = '\0';
}

#endif
```