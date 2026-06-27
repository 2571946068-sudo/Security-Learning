题目信息

来源：攻防世界
题目名称：GFSJ0257--4-2
题目类型：Misc（杂项）

我的解题思路
初步尝试：看到密文保留了空格和标点，像是英文，所以先尝试了凯撒密码在线解码，但没有得到有意义的结果。
判断密码类型：因为凯撒密码不行，而且每个字母的替换看起来是随机的，所以判断这是单表替换密码。
确定破解方法：单表替换密码的特点是，每个明文字母被唯一一个密文字母替代。由于英文中每个字母出现的频率不同（例如'e'出现最多），我们可以通过词频分析来破解。统计密文中每个字母出现的次数，频率最高的很可能对应明文的'e'或't'。

解题工具与过程
工具：我找到了一个基于词频分析的C++脚本
#include <iostream>
#include <map>
#include <algorithm>
#include <sstream>
#include <vector>
#include <string>
using namespace std;

bool cmpp(pair<string, int> a, pair<string, int> b)
{
	return a.second > b.second;
}
bool cmp(pair<char, int> a, pair<char, int> b)
{
	return a.second > b.second;
}
void print_letter_word(vector<pair<char,int>> v, vector<pair<string,int>> vv)
{
	cout << "============================================" << endl;
	// 打印排序后的vec，以及替换为的list
	cout << "字母频度为:" << endl;
	for(int i=0; i<v.size(); i++)
		// 按照从高到低的次序打印频度
		cout << '"' << v[i].first << '"' << ":" << v[i].second << ",";
	cout << endl;
	cout << "单词频度为:" << endl;
	for(int i=0; i<vv.size(); i++)
		// 按照从高到低的次序打印频度
		cout << '"' << vv[i].first << '"' << ":" << vv[i].second << ",";
	cout << endl;
}
void print_h(vector<pair<char,int>> v, string list)
{
	cout << "============================================" << endl;
	cout << "替换规则如下:" << endl;
	cout << "密文字母频度降序排布:";
	for(int i=0; i<v.size(); i++) cout << v[i].first;
	cout << endl;
 
	cout << "以上字符按规则替换为:";
	for(int i=0; i<26; i++) cout << list[i];
	cout << endl;
}
 
void print_cleartext(map<char, char> h, string cipher_text)
{
	cout << "============================================" << endl;
	// 利用h将密文替换为明文进行输出
	cout << "明文如下:" << endl;
	for(int i=0; i<cipher_text.size(); i++)
	{
		char temp = cipher_text[i];
 
		if(islower(temp)) // 小写字母
			cout << h[temp]; // 输出替换后字符
		else // 非小写字母
			cout << temp;
	}
}
 
int main()
{
	// 读入
	string cipher_text;
	getline(cin, cipher_text);
	// getchar();
 
 
	// 计算字母和单词频数
	map<char, int> letter_m;
	map<string, int> word_m;
 
	for(char i='a'; i<='z'; i++) letter_m[i]=0; // 初始化
 
	for(int i=0; i<cipher_text.size(); i++)
	{
		if(islower(cipher_text[i]))
			letter_m[cipher_text[i]]++;
	}
 
	stringstream ss(cipher_text);
	string word;
 
	while (ss >> word)
	{
        // 对于每个单词，如果map中已经存储了该单词，则将其出现次数加一；否则，在map中新增该单词并设置其出现次数为1。
        while(!islower(word[word.size()-1])) word = word.substr(0, word.size()-1); // 去除尾部的标点
 
        if (word_m.find(word) != word_m.end())
            word_m[word]++;
        else
            word_m[word] = 1;
    }
 
	vector<pair<char,int>>  v(letter_m.begin(),letter_m.end()); // 用letter_map构建v
	sort(v.begin(), v.end(), cmp); // 排序
 
	vector<pair<string, int>> vv(word_m.begin(),word_m.end()); // 用word_map构建vv
	sort(vv.begin(), vv.end(), cmpp); // 排序
 
	print_letter_word(v, vv);
 
	// 创建字母替换表h
	map<char, char> h;
 
	string list = "etonahlisrmydwugbcfkvpj000";
 
	for(int i=0; i<v.size(); i++)
		h[v[i].first] = list[i];
    print_h(v, list); // 输出替换表
 
	// 输出明文
	print_cleartext(h, cipher_text);
 
	// 返回
	return 0;
}

过程
将密文全部转为小写。
运行脚本，得到初步的字母替换表。
根据初步明文，进行人工调整。例如，猜测常见的单词如the、get、flag等，不断调换字母映射，直到明文变得可读。
最终得到完整的替换规则，解出明文。
最终
flag{classical-cipher_is_not_security_hs}
