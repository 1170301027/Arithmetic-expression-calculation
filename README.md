# Arithmetic-expression-calculation
/*
一元多项式<1170301027fs>
*/
#include <stdio.h>
#include <stdlib.h>

typedef struct ploy
{
	int coef;//系数
	int exp;//指数
	struct poly *link;//指向下一个地址的指针
}polynode,*polypointer;//链表建立，节点指针类型定义
//添加新节点
polypointer Attch(int c,int e,polypointer d)
{
    polypointer x;
    x = (polypointer)malloc(sizeof(polynode));//分配一个结构体空间给x，以便插入新节点
    x->coef = c;
    x->exp = e;
    d->link = x;
    return x;//相当于原表后面添加新节点，并返回新节点的地址
}
//比较数据并返回相应关系用于判断
char Compare(int e1,int e2)
{
    if(e1>e2) return '>';
    else if(e1<e2) return '<';
    else return '=';
}
//链表建立
polypointer InitPoly(int data[][2],int n)
{
    polypointer c,d;
    c = (polypointer)malloc(sizeof(polynode));//制造表头
    d = c;//指向表头
    for(int i = 0;i<n;i++)//按照从文件录入的数据进行链表建立
    {
        d = Attch(data[i][0],data[i][1],d);
    }
    d->link = NULL;//完善该表尾
    return c;
}
//排序(前到后，大到小)
void Sort(int data[][2],int n)
{
    int temp;
    for(int i = 0;i<n;i++)//基本的冒泡排序
        for(int j = i+1;j<n;j++)
    {
        if(data[j][1]>data[i][1])
        {
            temp = data[j][0];
            data[j][0] = data[i][0];
            data[i][0] = temp;
            temp = data[j][1];
            data[j][1] = data[i][1];
            data[i][1] = temp;
        }
    }
}
//多项式加法
polypointer PloyAdd(polypointer a, polypointer b)
{
	polypointer p, q, d, c;
	int y;//存储系数和
	p = a -> link;//存储前一个多项式第一个节点
	q = b -> link;//存储后一个多项式第一个节点
	c = (polypointer)malloc(sizeof(polynode));//分配新的表头
	d = c;//存储新节点的首地址
	while((p!=NULL)&&(q!=NULL))//表尾判断
    {
        switch(Compare(p->exp,q->exp))
        {
            case '='://判断指数相等时候，系数相加
                y = p->coef + q->coef;
                if(y) d = Attch(y,p->exp,d);
                p = p->link;
                q = q->link;
                break;
            case '>'://判断前项指数大时，将前项添加到新表后面，且前项指向下一个位置
                d = Attch(p->coef,p->exp,d);
                p = p->link;
                break;
            case '<'://判断后项指数大时，将后项添加到新表后面，且后项指向下一个位置
                d = Attch(q->coef,q->exp,d);
                q = q->link;
                break;
        }
    }
    //循环结束，若有未经遍历的项要添加到新表后面
    while(p!=NULL)
    {
        d = Attch(p->coef,p->exp,d);
        p = p->link;
    }
    while(q!=NULL)
    {
        d = Attch(q->coef,q->exp,d);
        q = q->link;
    }
    d->link = NULL;
    free(p);
    free(q);
    return c;
}
//多项式减法（利用加法，节省代码）
polypointer PloySub(polypointer a, polypointer b)
{
    polypointer c;
    c = b;//存储表头
    b = b->link;//指向第一个节点
    do//将被减数的所有系数变为其相反数，以便利用加法实现减法节省空间
    {
        b->coef = -b->coef;
        b = b->link;
    }while(b!=NULL);
    return PloyAdd(a,c);
}
//多项式乘法
polypointer PloyMul(polypointer a, polypointer b)
{
    int data[50][2];
    int i = 0;
    polypointer p, q, d, c;
    p = a -> link;//存储前一个多项式第一个节点
	q = b -> link;//存储后一个多项式第一个节点
	c = (polypointer)malloc(sizeof(polynode));//分配新的节点，存表头
	d = c;//存储新节点的首地址
	while(p!=NULL)
    {
        while(q!=NULL)
        {
            data[i][0] = p->coef*q->coef;//储存相乘后的系数
            data[i][1] = p->exp+q->exp;//储存相乘后的指数
            q = q->link;//指向下一个节点
            i++;
        }
        data[i][1] = 50;//以防万一
        q = b->link;//更新q节点
        p = p->link;//指向下一个节点
    }
    Sort(data,i);//从大到小排序指数和相应系数
    for(int j=0;j<i;j++)//将排序好的数据用于建立新链表
    {
        if(data[j][1] == data[j+1][1])//将排序后的相同指数的项合并
        {
            data[j+1][0] += data[j][0];//合并到后项，使得j向后走时方便合并后面相同项
        }
        else
            d = Attch(data[j][0],data[j][1],d);
    }
    d->link = NULL;//表尾
    free(p);
    free(q);
    return c;
}
//多项式除法
void PloyDiv(polypointer a, polypointer b)
{
    polypointer c1,c2,d1,d2,p,q,pt;
    polypointer temp,ntemp;
	p = a -> link;//存储前一个多项式第一个节点
	pt = (polypointer)malloc(sizeof(polynode));//添加一个额外空表头pt
	pt->link = p;//用pt给p一个新的表头
	q = b -> link;//存储后一个多项式第一个节点
	c1 = (polypointer)malloc(sizeof(polynode));//分配新的表头做商表
	d1 = c1;//存储新节点的首地址
    c2 = (polypointer)malloc(sizeof(polynode));//分配新的表头做余数表
	d2 = c2;//存储新节点的首地址
	ntemp = (polypointer)malloc(sizeof(polynode));//新表头，暂存
    temp = ntemp;
	while(p!=NULL && q!=NULL)
    {
        if(p->exp>=q->exp)//被除数最大次数大于等于除数
        {
            temp = Attch(p->coef/q->coef,p->exp - q->exp,temp);//首项除法所得结果建立新表
            temp->link = NULL;
            d1 = Attch(temp->coef,temp->exp,d1);//添加到商表中
            pt = PloySub(pt,PloyMul(b,ntemp));//用首项除以首项所得的商乘以除数，用被除数去减这个结果作为新的被除数
            p = pt->link;//更新节点
            temp = ntemp;//更新暂存表指针至表头
        }
        else//被除数最大次数小于除数，此时不够除，因而除数为余数表，商表为零
        {
            c1->link = p;
            c2->link = NULL;
            break;
        }
    }
    d1->link = NULL;//表尾
    c2->link = p;//余数表头->p
    //free(p);
    free(q);
    printf("商多项式：\n");
    OutputPoly(c1);
    printf("余数多项式：\n");
    OutputPoly(c2);
}
//从文件中读取数据并保存
int ReadFile1(int data[][2])
{
	FILE *fp;
	int n = 0;
	if ((fp = fopen("C:\\Users\\冯帅\\desktop\\1.txt", "r")) != NULL)//以只读方式打开文件并判断是否打开并输出相应语句
	{
		printf("the file 1 was opened\n");
		while(!feof(fp)){
		fscanf(fp, "%d %d",&data[n][0],&data[n][1]);
		n++;
		}
		printf("succeed to read to array\n");
	}
	else printf("the file 1 was not opened\n");
	fclose(fp);
	return n;
}
int ReadFile2(int data[][2])
{
	FILE *fp;
	int n = 0;
	if ((fp = fopen("C:\\Users\\冯帅\\desktop\\2.txt", "r")) != NULL)//以只读方式打开文件并判断是否打开并输出相应语句
	{
		printf("the file 2 was opened\n");
		while(!feof(fp)){
		fscanf(fp, "%d %d",&data[n][0],&data[n][1]);
		n++;
		}
		printf("succeed to read to array\n");
	}
	else printf("the file 2 was not opened\n");
	fclose(fp);
	return n;
}
//将数据写入文件并保存
void WriteFIle(char data[][2],int n)
{
	FILE *fp;
	int n0 = n-1;//为下步循环做准备，确保从大到小输入
	if ((fp = fopen("C:\\Users\\冯帅\\desktop\\1.txt", "w+")) != NULL)//以读写方式打开文件并判断是否打开并输出相应语句
	{
		printf("the file was opened\n");
		while(n--){
		fprintf(fp,"%d %d",data[n0-n][0],data[n0-n][1]);
		}
		printf("succeed to write to file\n");
	}
	else printf("the file was not opened\n");
	fclose(fp);
}
//计算多项式在x0处的值
float Calculate(float x0,polypointer a)
{
    polypointer b;
    a = a->link;
    float sum = a->coef;//记录结果，同时充当系数（v0 = an
    int n = a->exp;//存储最大幂指数
    b = a->link;
    while(n--)//秦九韶算法
    {
        if(b->exp == n)//下一项指数紧邻
        {
            sum = x0*sum + b->coef;//vi = v【i-1】*x0 + a【n-i】
            b = b->link;
        }
        else//下一项不紧邻
            sum = x0*sum;//vi = v【i-1】*x0 + 0
    }
    return sum;
}
//菜单
void Menu()
{
        printf("-------------------------------\n");
        printf("| 1.读取文件并保存            |\n");
        printf("| 2.写入文件并保存            |\n");
        printf("| 3.多项式加法                |\n");
        printf("| 4.多项式减法                |\n");
        printf("| 5.多项式乘法                |\n");
        printf("| 6.多项式除法                |\n");
        printf("| 7.计算多项式在x0处的值      |\n");
        printf("| 8.退出                      |\n");
        printf("-------------------------------\n");
        printf("请输入您想实现的操作:");
}
//输出多项式
void OutputPoly(polypointer a)
{
    polypointer p;
    p = a->link;
    if(p!=NULL)//判断是否是空表,若是则输出零，若不是则先给出第一项
    {
        printf("%dx^%d",p->coef,p->exp);
        p = p->link;
    }
    else
        printf("0");
    while(p!=NULL)//当不是空表时，逐项判断系数正负并输出
    {
        if(p->coef<0)
            printf("-%dx^%d",-p->coef,p->exp);
        else
            printf("+%dx^%d",p->coef,p->exp);
        p = p->link;
    }
    printf("\n");
}
//读取两个多项式（冗余部分）
void Read(polypointer* a,polypointer* b,int data[][2],int ndata[][2],int *n,int *n0)
{
    //指针操作是为了改变主函数中相应值，由于该操作在程序中四则运算都要运算一边，所以直接添加函数进来，方便操作和更改
    printf("分别从1,2文件中读取两个多项式进行运算：\n");
    *n = ReadFile1(data);*n0 = ReadFile2(ndata);
    Sort(data,*n);*a = InitPoly(data,*n);
    Sort(ndata,*n0);*b = InitPoly(ndata,*n0);
    OutputPoly(*a);OutputPoly(*b);
}
int main()
{
	int data[50][2],ndata[50][2];//两多项式系数指数存储
	int n,n0;//项数存储
	polypointer a,b,result;//ab用于存储操作多项式，result用于存储结果多项式
	float x0;
	int op;//存储键盘录入的选项
	while(1){
        Menu();
        do{//输入合法性判断
            scanf("%d",&op);
            if(op<=0 || op>=9) printf("您输入的数据不合法，请重新输入\n");
        }while(op<=0 || op>=9);
        switch(op)
        {
            case 1:n = ReadFile1(data);Sort(data,n);a = InitPoly(data,n);OutputPoly(a);break;
            case 2:WriteFIle(data,n);break;
            case 3:Read(&a,&b,data,ndata,&n,&n0);result = PloyAdd(a,b);OutputPoly(result);break;
            case 4:Read(&a,&b,data,ndata,&n,&n0);result = PloySub(a,b);OutputPoly(result);break;
            case 5:Read(&a,&b,data,ndata,&n,&n0);result = PloyMul(a,b);OutputPoly(result);break;
            case 6:Read(&a,&b,data,ndata,&n,&n0);PloyDiv(a,b);break;
            case 7:n = ReadFile1(data);Sort(data,n);a = InitPoly(data,n);
                   printf("请输入x0的值：");scanf("%f",&x0);printf("%f\n",Calculate(x0,a));
                   break;
            case 8:exit(0);
        }
	}
	return 0;
}
