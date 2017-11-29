#include<stdlib.h> 
#include<stdio.h>  
#include<math.h>  
#include<string.h>  
#include<time.h> 
#define MAXNUM 10      
#define MAXNUM 10 
#define NUMBER 20//最大物品数量     
#define TRUE 1     
#define FALSE 0   
typedef struct book  
{   
	long int starting;//借书日期  
	long int ending;// 应还日期   
	char bookinform[120];// 这里面存储书籍的描述信息  
	long int callnum;// 索书号  
	char bookname[30];   
	char writer[20];   
	int totalstorage, nowstorage;//书本的馆藏量 
}book;   

//本结构用于创建链表的二叉树 
typedef struct volume  
{   
	book books; 
	struct volume *lchild;   
	struct volume *rchild;  
}volume,*Btvolume;   

typedef struct libcard  
{   
	char userID[12];   
	char password[12];// 密码   
	char clientname[20];// 用户名  
	char usermessage[150];// 用户信息   
	book borrow[10];// 每本借书证限借书十本   
	struct libcard *next;  
}libcard; 

libcard *clients,*current_client=NULL;  

//函数声明 
void createBST(Btvolume *bst);  
void insertBST(Btvolume *bst,book *key);  
Btvolume searchBST(Btvolume bst,long int key);  
void GetPassword(char *str,int n);  
int registerer();  
int login();// 登录
void clientsev(volume **Btroot);// 客户（学生 
int check_client(libcard **client);  
void in_stor(volume **root);// 入库  
void backbook(Btvolume *root,long int callnum);// 还书 
int lend(Btvolume *root,long int callnum);// 借书，有学生发出请求   

struct Record//本结构用于保存每一次结果     
{ 
    int totalWeight;//本次结果的总价值 
    int goods[NUMBER];//本次结果对应的下标      
	struct Record *next;      
}; 
struct Record *headLink;     
struct Record result;     
int stack[NUMBER];     
int top; 
int weight[NUMBER];//保存物品重量的数组 
int value[NUMBER];//保存对应（下标相同）物品的价值     
int knapproblen(int n,int maxweight,int weight[]);     
void CreateHeadLink(void); 
struct Record *MallocNode(void);     
void InsertOneNode(struct Record *t);     
void GetResult(void);     
void ShowResult(void);  
main()
{     
	int a; 
    printf("---------------- 选择访问程序---------------- \n");  
	printf("---------------- 1.背包问题  ---------------- \n");  
	printf("---------------- 2.图书系统  ---------------- \n");  
	printf("---------------- 0.退出系统  ---------------- \n"); 
	printf("--------------------------------------------- \n"); 
	scanf("%d",&a);     
	if(a==1) 
        beibao();  
    else if(a==2)        
		tushuguanli();   
}   
	
	 //建立图书库第一本书 
void createBST(Btvolume *bst)  
{   
	book *key;     
	*bst=NULL;     
	key=(book *)malloc(sizeof(book));    
	key->starting=0;    
	key->ending=0;    
	printf("输入索书号：");    
	scanf("%ld",&(key->callnum));    
	if(key->callnum == 0)
	{
		 free(key);         
	}    
	printf("输入入库量：");    
	scanf("%d",&(key->totalstorage));    
	key->nowstorage=key->totalstorage;    
	printf("输入书本名：");    
	scanf("%s",&(key->bookname));    
	printf("输入著作名：");    
	scanf("%s",&(key->writer));    
	printf("输入书描述：");    
	scanf("%s",&(key->bookinform));          
	if(key!=NULL)    
	insertBST(bst,key);  
}

//二叉排序树的插入模块，采用递归算法实现
void insertBST(Btvolume *bst,book *key)  
{   
	Btvolume s;   
	if(*bst==NULL)// 递归结束条件   
	{    
		s=(Btvolume)malloc(sizeof(volume));     
		s->books.callnum = key->callnum; 
   		s->books.nowstorage = key->nowstorage;    
		s->books.totalstorage = key->totalstorage;    
		s->books.starting = key->starting;    
		s->books.ending = key->ending; 
   		strcpy(&(s->books.bookinform),&(key->bookinform));    
		strcpy(&(s->books.bookname),&(key->bookname));    
		strcpy(&(s->books.writer),&(key->writer)); 
    	s->lchild=NULL;    
		s->rchild=NULL;    
		*bst=s;   
	} 
  	else if(key->callnum < (*bst)->books.callnum)   
	{    
		insertBST(&((*bst)->lchild),key);   
	} 
  	else if(key->callnum > (*bst)->books.callnum)   
	{    
		insertBST(&((*bst)->rchild),key);   
	}   
	else 
	{   
	 	(*bst)->books.nowstorage+=key->nowstorage;    
		(*bst)->books.totalstorage+=key->totalstorage;   
	} 
} 
  //二叉排序树的查找算法，按照索书号为关键字进行二分查找 
Btvolume searchBST(Btvolume bst,long int key)  
{   
 	if(bst==NULL)  
  	return NULL; 
  	if(bst->books.callnum == key)    
	  	return bst;   
	if(bst->books.callnum < key)    
		return searchBST(bst->rchild,key);   
		return searchBST(bst->lchild,key);  
}  

//入库函数，用于增加书本库存， 函数无返回值 实现：1.排序二叉树的建立；
//2.排序二叉树的查找；3.排序二叉树的插入；4.排序二叉树的删除；
 
void in_stor(volume **root)  
{   
	Btvolume p,q;   
	book *key;    
	
	//输入数据   
	key=(book*)malloc(sizeof(book));   
	printf("输入索书号：");   
	scanf("%ld",&(key->callnum));   
	printf("输入入库量：");   
	scanf("%d",&(key->totalstorage));   
	key->nowstorage=key->totalstorage;   
	printf("输入书本名：");   
	scanf("%s",&(key->bookname));   
	printf("输入著作者：");
	scanf("%s",&(key->writer));   
	printf("输入书描述：");   
	scanf("%s",&(key->bookinform));   
	key->starting = 0;   
	key->ending = 0;    
	insertBST(root,key);  
}   

//借书模块，由学生发出请求，返回借书是否成功，若当前的可借阅量大于0并且学生当前借书量未超过限制，则借书成功。否则失败
 
int lend(Btvolume *root,long int callnum)  
{   
	int success=0;// 作为返回值，用于标记借阅是否成功 
	int i=0;   
	time_t t;   
	Btvolume temp;
	while(current_client->borrow[i].callnum>0)    
		i++;   
	if(i>=10)    
		printf("对不起，你借阅的书籍过多，请先还书!\n");   
	else   
	{    
		temp=searchBST(*root,callnum);    
		if(temp==NULL)     
			printf("对不起，本馆没有收藏此书!\n");    
		else if(temp->books.nowstorage > 0)    
		{     
			time(&t);     
			temp->books.nowstorage -= 1;     
			current_client->borrow[i].callnum = callnum;     
			current_client->borrow[i].starting = t;// 获取当前系统时间   
			current_client->borrow[i].ending = current_client->borrow[i].starting+2592000;     
			current_client->borrow[i].totalstorage = temp->books.totalstorage;     
			current_client->borrow[i].nowstorage = temp->books.nowstorage;     
			strcpy(current_client->borrow[i].bookinform , temp->books.bookinform);     
			strcpy(current_client->borrow[i].bookname , temp->books.bookname);     
			strcpy(current_client->borrow[i].writer , temp->books.writer);     
			success=1;      
			printf("您已经成功借书，欢迎下次光临\n");    
		}    
		else     
			printf("对不起，该书应被借完!\n");   
	}   
	return success;
}

//还书模块，由学生发出请求，每次只能还一本书，即便是两本同样的书也需要还两次，无返回值，还书后，相应库存量增加，同时学生的当前借阅量减少
//如果超期，返回超期天数
void backbook(Btvolume *root,long int callnum)  
{   
	Btvolume temp;   
	time_t t;   
	int i=0;   
	double overtime;    
	while(i<=10)
	{    
		if(current_client->borrow[i].callnum==callnum)     
			break;    
		i++;   
	}   
	if(i>=10)   
	{    
		printf("对不起，您并未借过此书!\n");    
		return;   
	}   
	temp=searchBST(*(root),callnum);   
	if(temp==NULL)   
	{    
		printf("您好，本馆并未藏有此书，因此你所还的书不是本馆\n");    
		return;   
	}   
	if(temp->books.nowstorage >= temp->books.totalstorage)   
	{    
		printf("对不起，本馆并未借出此书!\n");    
		return;   
	}   
	time(&t);   
	if( t > current_client->borrow[i].ending)   
	{    
		overtime=((t- current_client->borrow[i].ending)/43200)+1;    
		printf("对不起，您所借的书已经超期 %0.0f 天\n",overtime);   
	}      
	
	//还书后所做的处理  
	temp->books.nowstorage++;
	current_client->borrow[i].bookinform[0]='\0';   
	current_client->borrow[i].bookname[0]='\0';   
	current_client->borrow[i].callnum=0;   
	current_client->borrow[i].ending= 2004967296;   
	current_client->borrow[i].starting=0;   
	current_client->borrow[i].nowstorage=0;   
	current_client->borrow[i].totalstorage=0;   
	current_client->borrow[i].writer[0]='\0';    
	printf("书已经成功归还，欢迎下次光临!\n");  
} 

//注册模块 
int registerer()   
{
	libcard *client,*client_temp=NULL;    
	char temp[13];    
	int type,success=0;    
	int i;       
	while(1)
	{              
		while(1)
		{        	 
			client=(libcard*)malloc(sizeof(libcard));     
			client->next=NULL;      
			printf("------------按照要求填写有关注册信息-------------\n");      
			printf("用户ID:");      
			scanf("%s",&client->userID);       
			//检测是否为重复注册    
			client_temp=clients;      
			while(client_temp!=NULL)      
			{       
				if(!strcmp(client->userID , client_temp->userID))// 不允许相同ID，但允许相同密码      
					break;       
					client_temp=client_temp->next;      
			}      
			if(client_temp == NULL)//没有相同的ID     
			{      
			//注册成功后应对信息进行初始化      
				for(i=0;i<10;i++)
				{        
					client->borrow[i].bookinform[0] = '\0';        
					client->borrow[i].bookname[0] = '\0';        
					client->borrow[i].writer[0] = '\0';        
					client->borrow[i].callnum = 0;        
					client->borrow[i].starting = 0;        
					client->borrow[i].ending = 315360000;//将归还日期初始为无穷大      
					client->borrow[i].nowstorage = 0;        
					client->borrow[i].totalstorage = 0;       
				}              
				client->next=clients;
				clients=client;      
			}      
			else      
			{       
				printf("对不起，该ID已经被占用，注册失败!\n");       
				break;      
			}      
			while(1)      
			{       
				printf("密码:");       
				GetPassword(&client->password,12);       // scanf("%s",&client->password);       
				print("确认密码:");       
				GetPassword(&temp,12);       // scanf("%s",&temp);       
				if(!strcmp(client->password,temp))// 检测密码是否有误       
					break;       
				printf("您输入的密码有误，请重新输入\n");      
			}      
			printf("请输入用户名字");      
			scanf("%s",&client->clientname);      
			printf("请输入用户描述");      
			scanf("%s",&client->usermessage);          
				break;     
		}       
		printf("是否继续注册？(1.继续)(0.退出)");        
		scanf("%d",&type);        
		if(!type)              
			break;             
	}  
	return success;   
}  

//核对用户信息  
int check_client(libcard **client)  
{   
	int success=0;   
	libcard *temp=clients;   
	while(temp!=NULL )            //循环验证借书证 
	{    
		if(!strcmp((*client)->userID , temp->userID)&&!strcmp((*client)->password , temp->password))                       //如果有一个不对应，则跳入下面一个用户验证  
		{  	  
			free(*client);     
			*client=temp;     
			success=1;    
			break;    
		}    
		temp=temp->next;   
	} 
  return success; 
 } 
  //客户 
void clientsev(volume **Btroot)  
{   
	int a,b;   
	volume *temp;   
	libcard *p=NULL;// 用作临时指针    
	if(current_client==NULL)   
	{    
		printf("您还未登录，请登录\n");    
		login();    
		return;//不管是否成功登陆都将返回原处   
	} 
   	while(1)   
	{    
		system("cls");// 清屏
		printf("----------------- 图书管理系统 -----------------\n");   
		printf("-------------------客户用户---------------------\n"); 
		printf(" --------------- 1。图书借阅-------------------\n");   
		printf(" --------------- 2。图书归还-------------------\n");   
		printf(" --------------- 0。退出-----------------------\n"); 
		    scanf("%d",&a);    
		switch(a)    
		{     
   			case 1:     
			   	printf("输入索书号\n");     
				scanf("%d",&b);
				lend(Btroot,b); 
				break;    
			case 2:     
				printf("输入索书号\n");     
				scanf("%d",&b);     
				backbook(Btroot,b);     
				break;     
   			default:return;    
		} 
   		system("pause");// ////////////////////////// 
  	} 
 } 
  	
	  ////登录模块，返回登录的用户类型，用户必须先登录才能对图书进行操作 
	  
int login()  
{   
	libcard *temp_client=NULL;   
	int type,temp;   
	printf("-----------------图书管理系统----------------\n"); 
	printf("-----------------客户登录-----------------\n");   
	printf("确认按1：");   
	scanf("%d",&type);   
	while(1)   
	{    
		if(current_client!=NULL)
		{      
			printf("对不起，登录人数已满，您目前还不能登录\n");      
			break;     
		}     
		temp_client = (libcard *)malloc(sizeof(libcard));     
		printf("请输入借阅证号：");     
		scanf("%s",&temp_client->userID);     
		printf("请输入密码：");     
		GetPassword(&temp_client->password,12);     // scanf("%s",&temp_client->password);     
		if(check_client(&temp_client))//调用函数检查   
		{      
			current_client=temp_client;      
			return type;     
		}    
		else
		{ 
		 	printf("对不起，密码错误!\n");      
			free(temp_client);      
			current_client = NULL;      
			break;     
		}    
	}     
	printf("(0.退出)\n");    
	scanf("%d",&temp);    
	if(!temp)return 0;           
}
////////从键盘获取字符密码，显示为*，设定最大字符数为n 
void GetPassword(char *str,int n) 
{  
	int i=0;  
	char c;  
	while((c=getch())!=13&&i<n)//不回显 
	{   
		if(c!=8)   
		{    
			putch('*');//回显    
			str[i++]=c;   
		}   
		else    
			i--; 
		}  
		putch('\n');  
		str[i]='\0'; 
	}   
	int tushuguanli()  
	{   
		int a=0,b,i;   
		int cases=0,goornot;   
		book booker;   
		volume root,*Btroot;  
		printf("---------------欢迎访问图书管理系统--------------\n");     
		printf("---------------初次用系统建立书库   -------------\n"); 
		printf("---------------按照提示输入有关信息--------------\n");  
		
		///、、、、读取所有管理员和客户 
		createBST(&Btroot);// 在系统启动时调出记录   
		while(1)
		{ 
        	printf("--------------提示-----------------\n");   
			printf("----------按1，继续添加图书-----------\n");  
			printf("----------其他进入系统界面---------\n");  
			    scanf("%d",&i);   
			if(i==1)      
				in_stor(Btroot);   
			else      
				break;
		}     
		while(1)   
		{   
			system("cls");// //////////清屏   
			printf("-----------------图书管理系统-----------------\n");   
			printf("-----------------选择您的操作-----------------\n");   
			printf("------------------1.注册----------------------\n");   
			printf("------------------2。登录----------------------\n");   
			printf("------------------3.其他进入系统界面--------------\n");    
			cases=0;//    scanf("%d",&b);    
			switch(b)    
			{    
				case 1:     
					registerer();     
					break;
				case 2:
				    cases=login();    
					default:break;    
			} 
				    ///选择登录的用户，本系统必须先登录才能操作     
					printf("-------------------图书管理系统------------------\n");   
					printf("-------------------选择您的操作 ------------------\n");  
					printf(" -----------------1用户注册---------------------\n");   
					printf(" -----------------2用户登录---------------------\n");  
					printf(" -----------------3图书入库---------------------\n"); 
					printf(" -----------------4进入用户界面-----------------\n");  
					scanf("%d:",&b);    
					switch(b)    
					{    
						case 1:     
							registerer();
							break;             
						case 3:     
							in_stor(Btroot);     
							break;             
						case 2:     
							login();     
							break;    
						case 4:      
							clientsev(&Btroot);     
							default:break;            
					}        
					system("cls");// //清屏   
					printf("是否继续（1.继续）（0.退出）");    
					scanf("%d",&goornot);    
					if(!goornot)     
					break;   
		}           
				return 0;  
				
}   
