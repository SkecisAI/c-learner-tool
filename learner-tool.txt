#include<stdio.h>
#include<stdlib.h>
#include<string.h>

#define TRUE 1
#define FALSE 0
typedef enum{xiaobai=1,mengxin=3,xuetu=5,zhizhe=7,dashi=9,shuoshi=11,boshi=13,yanjiusheng=15,xuezhe=17,jiaoshou=19,xuexijia=21,zsxx=23} ranks;//用户阶层枚举型
struct usr_infor//用户学习信息结构体
{
    int level;//等级
    ranks rank;//阶层
    long grade;//分数
};
struct usr//用户信息结构体
{
    char name[10];
    int id;
    int password;
    struct usr_infor infor;
};
double tp_record[10000];//记录用户学习成果
struct usr simple={"aaa",1,111,{1,1,0}};//初始用户
struct usr admin={"admin",0,999,{1,1,0}};//初始管理员
struct usr All_usr[100];//用户成员数组
struct usr *temp_usr;//临时用户
FILE *usr_file;//用户分数文件
FILE *usr_pwd_file;//用户密码文件
FILE *helptxt;
FILE *info_record;
FILE *newrecord;
//                 基本函数                 //
double pow(double a,double b);//乘方函数
double sqrt_p(double num,double acu);//开方函数(数字，精确度)
double abs_p(double a);//绝对值函数
void clear();//清屏
void clean_cmd();//清理输入缓冲区
double average(double *nums,int n);//求平均数
double variance(double *nums,int n);//求方差
double sums(double *nums,int n);//求和
double reg_equation(double *nums,int n,char coe);//求回归方程
long num_resort(long num);//倒序一个数
int num_size(long num);//数字的长度
int num_getn(long num,int n);//字符数字提取
int num_ascii(int num);//获取数字的ASCII码
int chnu_to_innu(char num);//将字符型数字转换为整型数字
double detmin_value(double *dt,int n);//求行列式的值
//#####################量化函数#####################//
long a(int n)
{
    return 20*n-10;
}
long b(int n)
{
    return 10*n*n-20*n+50;
}
//##########################################//

void screenfetch(int xu);//界面打印
//void cmdMode();//命令模式
//###########################################//
//             函数数表功能                  //
//###########################################//
void FuC_list();     
void FuC_xN();//计算一元函数的数表
void fx_create();//函数的构造
void fx_print(double *fx,double max,double min,int mci,double dt);//函数打印
void x_statistics();//对一列数的操作
//###########################################//
//             矩阵运算功能
//###########################################//
void martrix();     
void martrix_determinant();//计算行列式的值
void martrix_equation();//线性方程组求解
double det_equtions(double *dt,int n);//克拉默法则
void martrix_multi();//矩阵的乘法运算
void martrix_ranks();//求矩阵的秩
void martrix_inverse();//求矩阵的逆
//###########################################//
//             量化学习管理系统:466行
//###########################################//
void study_manage();
void study_record(int usrid);//学习记录
void write_grades(long newgrades,int id);//写分数
void usr_inforREAD(int usrid);//读取用户的信息
void usr_inforWRITE(int usrid);//修改用户的信息
void usrscreen(int usrid);//绘制用户的个人界面
long study_rulers();//量化规则
void subject_print();//打印信计课程信息
int rulers_date(int n,int data);//量化规则
void put_data(int usrid);//从文件导入用户的数据:590行
long rulers_function(int x);//量化函数
//称号输出
int input(ranks *r,int lv);
void output(ranks r);
void write_data(int usrid);//写入文件
void regist();//注册账号
//###########################################//
//             帮助
//###########################################//
void help();
//###########################################//
//             用户账号管理
//###########################################//
void usr_list();
//###########################################//
//###########全局定义一些表示状态量的标志变量status########//
int firstpage=TRUE;//主界面状态
int FuC_page=FALSE;//函数界面
int martrix_page=FALSE;//矩阵界面
int study_page=FALSE;//学习管理界面
int rules_page=FALSE;//
int offline=TRUE;//在线状态
int r_able=FALSE;//是否可读
int w_able=FALSE;//是否可写
int newer=FALSE;//新用户
int reset=FALSE;//重置
int pwd_reset=FALSE;//重置密码
int lv_up=FALSE;//升级
int record=FALSE;
//#########################################//
//主函数
int main(void)
{
    int i=0,j=0,cmd=0,usrpwd=0,n,k=0;
    char data,name[10]="";
//    All_usr[0]=admin;//管理员数据写入
//    All_usr[1]=simple;//普通初始用户写入
    for(i=0;i<100;i++)
    {
        All_usr[i].id=i;
    }
    usr_pwd_file=fopen(".usr_pwd.txt","r");//读入文件初始化用户名和密码
    //文件格式为:i(id):#name#Password!
    for(i=0;i<100;i++)
    {
        usrpwd=0;
        j=0;
        while(1)
        {
            data=fgetc(usr_pwd_file);//获取第一个字符i
            if(data=='$')
            {
                while(data!=':')
                {
                    data=fgetc(usr_pwd_file);
                }
                data=fgetc(usr_pwd_file);//读第一个#
                data=fgetc(usr_pwd_file);
                if(data=='#')//说明用户不存在
                {
                    All_usr[i].password=0;
                    break;//exit while(1)读下一个i
                }
                All_usr[i].name[j]=data;//j=0,data!='#'
                while(data!='#')
                {
                    data=fgetc(usr_pwd_file);
                    if(data!='#')
                    {
                        ++j;
                        All_usr[i].name[j]=data;
                    }
                }
//                All_usr[i].name[j]='!';//名字结束符号
                for(j=0;j<3;j++)
                {
                    data=fgetc(usr_pwd_file);
                    usrpwd+=chnu_to_innu(data)*pow(10,2-j);
                }
                All_usr[i].password=usrpwd;
                break;
            }
        }
    }
    fclose(usr_pwd_file);
    for(i=0;i<100;i++)
    {
        All_usr[i].infor.grade=0;
        All_usr[i].infor.level=1;
    }
    clear();
    while(1)
    {
        for(i=0;i<10000;i++) tp_record[i]=0;
        if (firstpage==TRUE)
        {
            cmd=0;
            screenfetch(0);
            scanf("%d",&cmd);
            getchar();
            clean_cmd();
            switch(cmd)
            {
                case 0:
                {
                    clear();
                    clean_cmd();
                    printf("Mainmode:Invalid command!\n");
                    break;
                }
                case 1:
                {
                    FuC_page=TRUE;
                    firstpage=FALSE;//关闭主界面
                    clear();
                    FuC_list();//调用生成函数数表函数
                    break;
                }
                case 2:
                {
                    martrix_page=TRUE;
                    firstpage=FALSE;//关闭主界面
                    clear();
                    martrix();//调用矩阵运算界面
                    break;
                }
                case 3:
                {
                    firstpage=FALSE;//关闭主界面
                    study_page=TRUE;
                    clear();
                    study_manage();//调用学习管理系统
                    break;
                }
                case 4:
                {
                    clear();
                    firstpage=FALSE;
                    usr_list();//用户查看
                    firstpage=TRUE;
                    break;
                }
                case 5:
                {
                    clear();
                    firstpage=FALSE;
                    newer=TRUE;
                    regist();//注册账号
                    firstpage=TRUE;
                    newer=FALSE;
                    break;
                }
                case 6:
                {
                    clear();
                    firstpage=FALSE;
                    help();
                    firstpage=TRUE;
                    break;
                }
                case 7: return 0;//退出
                default:
                {
                    clear();
                    printf("Mainmode:Invalid command!\n");
                    break;
                }
            }
        }
    }
    return 0;
}
void clear()
{
    system("clear");
}
void clean_cmd()
{
    setbuf(stdin,NULL);
}
//########################################################//
void screenfetch(int xu)
{ 
    switch(xu)
    {
        case 0:
        {
            if (firstpage==TRUE)
            {
                printf("------------------------------\n");
                printf("|◤        学习工具箱 v1.0   ◥|\n");
                printf("|----------------------------|\n");
                printf("|        $-1.函数数表        |\n");
                printf("|        $-2.矩阵运算        |\n");
                printf("|        $-3.学习管理*       |\n");
                printf("|        $-4.用户列表*       |\n");
                printf("|        $-5.注册账号        |\n");
                printf("|        $-6.帮助            |\n");
                printf("|◣       $-7.退出           ◢|\n");
                printf("------------------------------\n");
                printf("Please enter your choice(1-7):");
            }
            break;
        }
        case 1:
        {
            if (FuC_page==TRUE)
            {
                printf("------------------------------\n");
                printf("|         >函数数表<         |\n");
                printf("|----------------------------|\n");
                printf("|        1.一元N次函数       |\n");
                printf("|        2.X统计分析         |\n");
                printf("|        3.返回              |\n");
                printf("|                            |\n");
                printf("|                            |\n");
                printf("|                            |\n");
                printf("|                            |\n");
                printf("------------------------------\n");
                printf("Please enter your choice(1-3):");
            }
            break;
        }           
        case 2:
        {
            if (martrix_page==TRUE)
            {
                printf("------------------------------\n");
                printf("|         >矩阵运算<         |\n");
                printf("|----------------------------|\n");
                printf("|        1.行列式求值        |\n");
                printf("|        2.线性方程组求解    |\n");
                printf("|        3.矩阵的乘法        |\n");
                printf("|        4.矩阵的秩          |\n");
                printf("|        5.矩阵的逆          |\n");
                printf("|        6.返回              |\n");
                printf("|                            |\n");
                printf("------------------------------\n");
                printf("Please enter your choice(1-5):");
            }
            break;
        }
    }
}
//#########################################################//
void FuC_list()//函数数表功能实现
{
    int cmd;
    while(1)
    {
        if (FuC_page==TRUE&&firstpage==FALSE)
        {
            screenfetch(1);
            cmd=0;
            scanf("%d",&cmd);
            getchar();
            clean_cmd();
            switch(cmd)
            {
                case 1:
                {
                    FuC_page=FALSE;
                    clear();
                    fx_create();
                    break;
                }
                case 2:
                {
                    FuC_page=FALSE;
                    clear();
                    x_statistics();
                    break;
                }
                case 3:
                {
                    firstpage=TRUE;
                    FuC_page=FALSE;
                    clear();
                    break;
                }
                default:
                {
                    clear();
                    clean_cmd();
                    printf("FuCmode:Invalid command!\n");
                    break;
                }
            }
        }
        if (cmd==3) break;
    }
}
void fx_create()//一元函数的创建
{
    double max,maxlim,minlim,xi,delta;
    int ci,i;
    double *fc=(double*)malloc(100*sizeof(double));
    for(i=0;i<100;i++) *(fc+i)=0;
    printf("请输入该一元函数的最高次:");
    scanf("%lf",&max);
    printf("请依次输入各个项的系数与次数(如:x2 1 2):\n");
    do 
    {
        scanf("%lf %d",&xi,&ci);
        *(fc+ci)=xi;
    }while(ci!=0);
    printf("请输入X的下限：");
    scanf("%lf",&minlim);
    printf("请输入X的上限：");
    scanf("%lf",&maxlim);
    printf("请输入X的增长步长：");
    scanf("%lf",&delta);
    clear();
    fx_print(fc,maxlim,minlim,max,delta);//调用函数的输出
    clean_cmd();
    free(fc);
    printf("Press Enter to continue...");
    getchar();
    clear();
    FuC_page=TRUE;
}
void fx_print(double *fx,double max,double min,int mci,double dt)//函数的输出
{
    double x=0,y=0;
    int i;
    x=min;
    printf("\n            您输入的函数信息为:\n");
    printf("     ");
    printf("f(x)=");
    for(i=0;i<=mci;i++)
    {
        if(i==0)
        {
            if(*(fx+mci-i)==1) printf("x%d",mci-i);
            else if(*(fx+mci-i)==0) printf("");
            else if(*(fx+mci-i)==-1) printf("-x%d",mci);
            else printf("%dx%d",(int)*(fx+mci-i),mci-i);
        }
        else if(i==mci-1&&mci!=1)
        {
            if(*(fx+mci-i)==1) printf("+x");
            else if(*(fx+mci-i)==0) printf("");
            else if(*(fx+mci-i)==-1) printf("-x");
            else if(*(fx+mci-i)<0) printf("%dx",(int)*(fx+mci-i));
            else printf("+%dx",(int)*(fx+mci-i));
        }
        else if(i==mci)
        {
            if(*fx==1) printf("+1");
            else if(*fx==0) printf("");
            else if(*fx==-1) printf("-1");
            else if(*fx<0) printf("%d",(int)*fx);
            else printf("+%d",(int)*fx);
        }
        else
        {
            if(*(fx+mci-i)==1) printf("+x%d",mci-i);
            else if(*(fx+mci-i)==0) printf("");
            else if(*(fx+mci-i)==-1) printf("-x%d",mci-i);
            else if(*(fx+mci-i)<0) printf("%dx%d",(int)*(fx+mci-i),mci-i);
            else printf("+%dx%d",(int)*(fx+mci-i),mci-i);
        }
    }
    printf("    x属于[%d,%d]  \n     △x=%lf\n",(int)min,(int)max,dt);
    printf("       x           -       f(x)\n");
    do
    {
        y=0;
        for(i=0;i<42;i++) printf("-");
        printf("\n");
        printf("       %lf    |",x);
        for(i=0;i<=max;i++)
        {

            y=y+*(fx+i)*pow(x,i);
        }
        printf("       %lf",y);
        x=x+dt;
        printf("\n");
    }while(x<=max);
    for(i=0;i<42;i++) printf("-");
    printf("\n");
}
void x_statistics()
{
    int n=1;
    double x[1000];//从1开始存入
    char cmd;
    printf("输入您的数据:\n");
    while(1)
    {
        scanf("%lf",&x[n]);
        cmd=getchar();
        if(cmd=='\n') break;
        ++n;
    }
    printf("--------------------------------------------------------\n");
    printf("这组数的一系列特征值为:\n");
    printf("和:%lf\n",sums(x,n));
    printf("平均数为:%lf\n",average(x,n));
    printf("方差为:%lf\n",variance(x,n));
    printf("标准差为:%lf\n",sqrt_p(variance(x,n),0.000001));
    printf("回归方程为:y`=");
    if(reg_equation(x,n,'b')==1) printf("");
    else if(reg_equation(x,n,'b')==-1) printf("-");
    else printf("%lfx`",reg_equation(x,n,'b'));
    if(reg_equation(x,n,'a')==0) printf("\n");
    else if(reg_equation(x,n,'a')>0) printf("+%lf\n",reg_equation(x,n,'a'));
    else printf("%lf\n",reg_equation(x,n,'a'));
    printf("---------------------------------------------------------\n");
    printf("Press Enter to continue...");
    cmd=getchar();
    clear();
    clean_cmd();
    FuC_page=TRUE;
}
double average(double *nums,int n)//平均数的计算
{
    double sum=0,ave=0;
    int i=1;
    for(i=1;i<=n;i++) sum=sum+*(nums+i);
    ave=sum/n;
    return ave;
}
double sums(double *nums,int n)//一列数的求和
{
    double sum=0;
    int i=1;
    for(i=1;i<=n;i++) sum=sum+*(nums+i);
    return sum;
}
double variance(double *nums,int n)//方差的计算
{
    double ave=0,var=0;
    int i=1;
    ave=average(nums,n);
    for(i=1;i<=n;i++)
    {
        var=var+pow((*(nums+i)-ave),2);
    }
    var=var/n;
    return var;
}
double reg_equation(double *nums,int n,char coe)//回归方程的计算
{
    double _b=0,_a=0,x_i[1000],x_ave=0,y_ave=0,xy[1000],xx[1000];
    double xy_sum=0,xx_sum=0;
    int i=1;
    for(i=1;i<=n;i++) x_i[i]=i;
    x_ave=average(x_i,n);
    y_ave=average(nums,n);
    for(i=1;i<=n;i++) xy[i]=x_i[i]*(*(nums+i));
    xy_sum=sums(xy,n);
    for(i=1;i<=n;i++) xx[i]=pow(x_i[i],2);
    xx_sum=sums(xx,n);
    _b=(xy_sum-n*x_ave*y_ave)/(xx_sum-n*pow(x_ave,2));
    _a=y_ave-_b*x_ave;
    if(coe=='a') return _a;
    else return _b;
}
double sqrt_p(double num,double acu)//开方计算
{
    double sqr=0,delta=1,a=0,b=1,tp_a=0;
    while(abs_p(a-b)>=acu)
    {
        while(pow(a,2)<=num)
        {
            if(pow(a,2)==num) return a;
            tp_a=a;
            a+=delta;
        }
        b=a;
        a=tp_a;
        delta/=10;
    }
    sqr=a;
    return sqr;
}
double pow(double a,double b)//乘方函数
{
    double temp=0;
    if(b<0)
    {
        b=0-b;
        temp=a*pow(a,b-1);
        return 1/temp;
    }
    if (b==1) return a;
    else if(a==0) return 0;
    else if(b==0) return 1;
    else return a*pow(a,b-1);
}
double abs_p(double a)//绝对值函数
{
    if (a<0) return -a;
    else return a;
}
//#####################################################//
void martrix()//矩阵运算功能实现
{
    int cmd;
    while(1)
    {
        if (martrix_page==TRUE&&firstpage==FALSE)
        {
            cmd=0;
            screenfetch(2);
            scanf("%d",&cmd);
            getchar();
            switch(cmd)
            {
                case 1://行列式求值
                {
                    clear();
                    martrix_page=FALSE;
                    martrix_determinant();
                    martrix_page=TRUE;
                    break;
                }
                case 2://线性方程组求解
                {
                    clear();
                    martrix_page=FALSE;
                    martrix_equation();
                    martrix_page=TRUE;
                    clean_cmd();
                    break;
                }
                case 3://矩阵的乘法运算
                {
                    clear();
                    clean_cmd();
                    martrix_page=FALSE;
                    martrix_multi();
                    martrix_page=TRUE;
                    break;
                }
                case 4://矩阵的秩
                {
                    clear();
                    clean_cmd();
                    martrix_page=FALSE;
                    martrix_ranks();
                    martrix_page=TRUE;
                    break;
                }
                case 5://矩阵的逆
                {
                    clear();
                    clean_cmd();
                    martrix_page=FALSE;
                    martrix_inverse();
                    martrix_page=TRUE;
                    break;
                }
                case 6:
                {
                    martrix_page=FALSE;
                    firstpage=TRUE;
                    clear();
                    break;
                }
                default:
                {
                    clear();
                    clean_cmd();
                    printf("Martrixmode:Invalid command!\n");
                    break;
                }
            }
        }
        if (cmd==6) break;
    }
}
void martrix_inverse()//矩阵的逆
{
    double result=1,det[100][100],e_det[100][100],cmpx[100][100],t;
    double temp[100];
    int i,j,k,n=1;
    char end;
    for(i=0;i<100;i++)//弃用数组的0行和0列
    {
        det[i][0]=0;
        det[0][i]=0;
    }
    for(i=1;i<100;i++) e_det[i][i]=1;//单位矩阵
    printf("请输入您的矩阵:\n");
    while(1)//获得矩阵的大小
    {
        scanf("%lf",&det[1][n]);
        end=getchar();
        if(end=='\n') break;
        ++n;
    }
    for(i=2;i<=n;i++)//输入矩阵
    {
        for(j=1;j<=n;j++) scanf("%lf",&det[i][j]);
    }
    getchar();
    clean_cmd();
    for(i=1;i<=n;i++)
    {
        for(j=1;j<=n;j++)
        {
            cmpx[i][j]=det[i][j];//矩阵代入
        }
    }
    for(i=1;i<n;i++)
    {
        if(cmpx[i][1]==0)
        {
            for(j=1;j<=2*n;j++)
            {
                temp[j]=cmpx[i+1][j];//下一行给temp
                cmpx[i+1][j]=cmpx[i][j];//当前行给下一行
                cmpx[i][j]=temp[j];//temp给当前行
            }
        }
    }
    for(i=1;i<=n;i++)
    {
        for(j=1;j<=n;j++)
        {
            cmpx[i][n+i]=e_det[i][i];//对角单位矩阵代入
        }
    }
    for(i=1;i<=n;i++)//对复合矩阵进行初等行变换 
    {
        for(j=i+1;j<=n;j++)
        {
            if(cmpx[i][i]==0) break;
            t=cmpx[j][i]/cmpx[i][i];
            for(k=i;k<=2*n;k++)
            {
                cmpx[j][k]-=t*cmpx[i][k];
            }
        }
    }//获得下三角
    for(i=1;i<=n;i++) result*=cmpx[i][i];
    if(result==0)
    {
        printf("----------------------------------------\n");
        printf("该矩阵不可逆!\n");
        printf("----------------------------------------\n");
        printf("Press Enter to continue...");
        getchar();
        clear();
        clean_cmd();
        return;
    }
    for(i=n;i>1;i--)//逆向循环获得上三角
    {
        for(j=i-1;j>=1;j--)
        {
            t=cmpx[j][i]/cmpx[i][i];
            for(k=i;k<=2*n;k++)
            {
                cmpx[j][k]-=t*cmpx[i][k];
            }
        }
    }
    for(i=1;i<=n;i++)
    {
        t=cmpx[i][i];/////////
        for(j=1;j<=2*n;j++)
        {
            cmpx[i][j]=cmpx[i][j]/t;//i,j会与i,i重复,i,i就会一直为1
        }
    }
    printf("----------------------------------------\n");
    printf("该矩阵的逆为:\n");
    for(i=1;i<=n;i++)
    {
        printf("|");
        for(j=n+1;j<=2*n;j++)
        {
            printf("%lf ",cmpx[i][j]);
        }
        printf("\n");
    }
    printf("----------------------------------------\n");
    printf("Press Enter to continue...");
    getchar();
    clean_cmd();
    clear();
}
void martrix_ranks()
{
    double det_A[100][100],tp=0,t;
    int i=1,j=1,k=1,n=1,s=1,temp=1,tp_size,cont=0,dot_set=0;
    int one=FALSE,dot=FALSE,rk=0,zero=0;
    char cmd='a',c[20]="";
    printf("请输入您的矩阵(两次回车=矩阵结束):\n");
    for(i=1;i<100;i++)
    {
        for(j=0;j<100;j++) det_A[i][j]=0;
    }
//对第一个的处理
    while(1)//记录行数
    {
        scanf("%lf",&det_A[1][n]);
        cmd=getchar();
        if(cmd=='\n') break;
        ++n;
    }
    //下一行2
    while(1)
    {
        cmd=getchar();
        if(cmd!='\n')
        {
            ++s;
            c[1]=cmd;//取第二行第一个数字的第一个字符
            cont=2;
            cmd=getchar();//读取第二个字符
            if(cmd==32)//如果读到的是空格
            {
                det_A[s][1]=chnu_to_innu(c[1]);
                one=FALSE;
                scanf("%lf",&det_A[s][2]);
            }
            else
            {
                one=TRUE;
                c[2]=cmd;
                while(cmd!=32)
                {
                    cmd=getchar();
                    if(cmd!=32)
                    {
                        ++cont;
                        if(cmd=='.')
                        {
                            dot=TRUE;
                            c[cont]='.';
                            dot_set=cont;
                        }
                        else
                        {
                            c[cont]=cmd;
                        }
                    }
                }
                dot=FALSE;
                if(dot_set==0)
                {
                    for(i=1;i<=cont;i++)
                    {
                        det_A[s][1]+=chnu_to_innu(c[i])*pow(10,cont-i);
                    }
                }
                else
                {
                    for(i=1;i<=dot_set-1;i++)//整数部分
                    {
                        det_A[s][1]+=chnu_to_innu(c[i])*pow(10,dot_set-1-i);
                    }
                    for(i=1;i<=cont-dot_set;i++)//小数部分
                    {
                        det_A[s][1]+=chnu_to_innu(c[dot_set+i])*pow(10,0-i);
                    }
                }
            }
            if(one==TRUE)
            {
                j=2;
                while(1)
                {
                    scanf("%lf",&det_A[s][j]);
                    cmd=getchar();
                    if(cmd=='\n') break;
                    ++j;
                }
            }
            else
            {
                j=3;
                while(1)
                {
                    scanf("%lf",&det_A[s][j]);
                    cmd=getchar();
                    if(cmd=='\n') break;
                    ++j;
                }
            }
        }
        else break;
    }
    clean_cmd();
    for(i=1;i<s;i++)//对行列式进行初等行变换
    {
        for(j=i+1;j<=n;j++)
        {
            if(det_A[i][i]==0) break;
            t=det_A[j][i]/det_A[i][i];
            for(k=i;k<=n;k++)
            {
                det_A[j][k]-=t*det_A[i][k];
            }
        }
    }
    rk=s;
    for(i=1;i<=s;i++)
    {
        for(j=1;j<=n;j++)
        {
            if(det_A[i][j]==0) ++zero;
        }
        if(zero==n) --rk;
    }
    printf("-----------------------------------------\n");
    printf("该矩阵的秩为:  %d\n",rk);
    printf("Press Enter to continue...");
    getchar();
    clear();
}
void martrix_multi()//矩阵的乘法运算
{
    double det_A[100][100],det_B[100][100],det_C[100][100],tp=0;
    int i=1,j=1,k=1,n=1,s=1,temp=1,tp_size,cont=0,dot_set=0;
    int one=FALSE,dot=FALSE;
    char cmd='a',c[20]="";
    printf("          矩阵的乘法运算(A x B = C)\n");
    printf("----------------------------------------\n");
    printf("输入矩阵 A (两次回车=矩阵结束):\n");
    for(i=1;i<100;i++)
    {
        for(j=0;j<100;j++) det_A[i][j]=0;
    }
//对第一个的处理
    while(1)//记录行数
    {
        scanf("%lf",&det_A[1][n]);
        cmd=getchar();
        if(cmd=='\n') break;
        ++n;
    }
    //下一行2
    while(1)
    {
        cmd=getchar();
        if(cmd!='\n')
        {
            ++s;
            c[1]=cmd;//取第二行第一个数字的第一个字符
            cont=2;
            cmd=getchar();//读取第二个字符
            if(cmd==32)//如果读到的是空格
            {
                det_A[s][1]=chnu_to_innu(c[1]);
                one=FALSE;
                scanf("%lf",&det_A[s][2]);
            }
            else
            {
                one=TRUE;
                c[2]=cmd;
                while(cmd!=32)
                {
                    cmd=getchar();
                    if(cmd!=32)
                    {
                        ++cont;
                        if(cmd=='.')
                        {
                            dot=TRUE;
                            c[cont]='.';
                            dot_set=cont;
                        }
                        else
                        {
                            c[cont]=cmd;
                        }
                    }
                }
                dot=FALSE;
                if(dot_set==0)
                {
                    for(i=1;i<=cont;i++)
                    {
                        det_A[s][1]+=chnu_to_innu(c[i])*pow(10,cont-i);
                    }
                }
                else
                {
                    for(i=1;i<=dot_set-1;i++)//整数部分
                    {
                        det_A[s][1]+=chnu_to_innu(c[i])*pow(10,dot_set-1-i);
                    }
                    for(i=1;i<=cont-dot_set;i++)//小数部分
                    {
                        det_A[s][1]+=chnu_to_innu(c[dot_set+i])*pow(10,0-i);
                    }
                }
            }
            if(one==TRUE)
            {
                j=2;
                while(1)
                {
                    scanf("%lf",&det_A[s][j]);
                    cmd=getchar();
                    if(cmd=='\n') break;
                    ++j;
                }
            }
            else
            {
                j=3;
                while(1)
                {
                    scanf("%lf",&det_A[s][j]);
                    cmd=getchar();
                    if(cmd=='\n') break;
                    ++j;
                }
            }
        }
        else break;
    }
//对第一个的处理
    clean_cmd();
    printf("----------------------------------------\n");
    printf("输入矩阵 B:\n");//s为列数 n为行数
    while(1)
    {
        scanf("%lf",&det_B[1][temp]);
        cmd=getchar();
        if(cmd=='\n') break;
        ++temp;
    }
    if(temp!=s)
    {
        clear();
        clean_cmd();
        printf("您的矩阵B不符合运算规则!\n");
        printf("----------------------------------------\n");
        martrix_multi();
        return;
    }
    clean_cmd();
    for(i=2;i<=n;i++)
    {
        for(j=1;j<=s;j++) scanf("%lf",&det_B[i][j]);
    }
    clean_cmd();
    printf("\n");
    printf("----------------------------------------\n");
    for(i=1;i<=s;i++)//A的行
    {
        for(j=1;j<=s;j++)//B的列
        {
            tp=0;
            for(k=1;k<=n;k++)//相乘的元素个数
            {
                tp+=det_A[i][k]*det_B[k][j];
            }
            det_C[i][j]=tp;
        }
    }
    printf("Result of A x B:\n");
    for(i=1;i<=s;i++)
    {
        for(j=1;j<=s;j++) printf("%lf  ",det_C[i][j]);
        printf("\n");
    }
    for(i=1;i<=40;i++) printf("-");
    printf("\nPress Enter to continue...");
    getchar();
    clear();
    clean_cmd();
}
void martrix_determinant()//行列式求值
{
    double result=1,det[100][100],t;
    int i,j,k,n=1;
    char end;
    for(i=0;i<100;i++)//弃用数组的0行和0列
    {
        det[i][0]=0;
        det[0][i]=0;
    }
    printf("请输入您的行列式:\n");
    while(1)//获得行列式的大小
    {
        scanf("%lf",&det[1][n]);
        end=getchar();
        if(end=='\n') break;
        ++n;
    }
    for(i=2;i<=n;i++)//输入行列式
    {
        for(j=1;j<=n;j++) scanf("%lf",&det[i][j]);
    }
    for(i=1;i<n;i++)//对行列式进行初等行变换
    {
        for(j=i+1;j<=n;j++)
        {
            if(det[i][i]==0) break;
            t=det[j][i]/det[i][i];
            for(k=i;k<=n;k++)
            {
                det[j][k]-=t*det[i][k];
            }
        }
    }
    clear();
    printf("该行列式的上三角形式:\n");
    for(i=1;i<=n;i++)
    {
        for(j=1;j<=n;j++) printf("%lf  ",det[i][j]);
        printf("\n\n");
    }
    printf("--------------------------------\n");
    for(i=1;i<=n;i++) result*=det[i][i];
    printf("该行列式的值为:%lf\n",result);
    printf("--------------------------------\n");
    printf("Press Enter to continue...");
    clean_cmd();
    end=getchar();
    clear();
}
void martrix_equation()//求解线性方程组
{
    int i,j,k,n=1,num=1;
    double au_mar[12][12],det[12][12],det_j[12][12],temp;
    double det_value=1;
    double x[12]={1,1,1,1,1,1,1,1,1,1,1,1};
    char cmd;
    printf("请输入线性方程组的增广矩阵:\n");
    while(1)
    {
        scanf("%lf",&au_mar[1][n]);
        cmd=getchar();
        if(cmd=='\n') break;
        ++n;
    }
    for(i=2;i<=n-1;i++)//获得线性方程组的增广矩阵
    {
        for(j=1;j<=n;j++) scanf("%lf",&au_mar[i][j]);
    }
    cmd=getchar();
    clean_cmd();
    for(i=1;i<=n-1;i++)//获得线性方程组的系数矩阵
    {
        for(j=1;j<=n-1;j++) det[i][j]=au_mar[i][j];
    }
    for(i=1;i<=n-2;i++)//得到系数矩阵行列式的值
    {
        for(j=i+1;j<=n-1;j++)
        {
            temp=det[j][i]/det[i][i];
            for(k=1;k<=n-1;k++)
            {
                det[j][k]-=temp*det[i][k];
            }
        }
    }
    for(i=1;i<=n-1;i++) det_value*=det[i][i];
    if(det_value==0) 
    {
        printf("该方程无解或没有唯一解！\n");
        printf("按回车已退出.....\n");
        cmd=getchar();
        clear();
        return;
    }
    while(num<=n-1)
    {
        for(i=1;i<=n-1;i++)//完成对dj行列式的赋值
        {
            for(j=1;j<=n-1;j++)
            {
                det_j[i][j]=au_mar[i][j];
                det_j[i][num]=au_mar[i][n];
            }
        }
        for(i=1;i<=n-2;i++)
        {
            for(j=i+1;j<=n-1;j++)
            {
                temp=det_j[j][i]/det_j[i][i];
                for(k=1;k<=n-1;k++)
                {
                    det_j[j][k]-=temp*det_j[i][k];
                }
            }
        }
        for(i=1;i<=n-1;i++) x[num]*=det_j[i][i];
        ++num;
    }
    printf("-----------------------------\n");
    printf("方程组的所有解为:\n");
    for(i=1;i<=n-1;i++) printf("x%d = %lf\n",i,x[i]/det_value);
    printf("-----------------------------\n");
    printf("回车键退出....");
    cmd=getchar();
    clear();
}
//#######################################################//
//                      学习管理系统模块                 //
void study_manage()
{
    char usr_name[10];
    int usr_id,usr_pwd,i=3,cmd=0;
    while(1)
    {
        
        if(offline==TRUE)
        {
            printf("       请输入您的信息:\n");
            while(1)
            {
                printf("您的用户名:");
                scanf("%s",usr_name);
                clean_cmd();
                printf("您的ID:");
                scanf("%d",&usr_id);
                printf("您的密码:");
                scanf("%d",&usr_pwd);
                if(i==0)
                {
                    clear();
                    firstpage=TRUE;
                    study_page=FALSE;
                    return;
                }
                if(usr_id<0||usr_id>99)
                {
                    clear();
                    printf("你的输入有误!!!\n");
                    --i;
                    continue;
                }
                if(strcmp(usr_name,All_usr[usr_id].name)==0&&usr_pwd==All_usr[usr_id].password&&usr_id>=0&&usr_id<=99)
                {
                    clear();
                    printf("登录成功!\n");
                    temp_usr=&All_usr[usr_id];
                    put_data(temp_usr->id);
                    study_record(temp_usr->id);
                    offline=FALSE;
                    break;
                }
                else 
                {
                    printf("用户名或密码错误!您还有%d次机会\n",i);
                    clean_cmd();
                    --i;
                    continue;
                }
            }
        }
        record=FALSE;
//        study_record(temp_usr->id);
//        printf("uuu\n");
        if(study_page==TRUE&&firstpage==FALSE)
        {
            usrscreen(temp_usr->id);
            scanf("%d",&cmd);
            getchar();
            switch(cmd)
            {
                case 1://读取用户的信息
                {
                    study_page=FALSE;
                    r_able=TRUE;
                    w_able=FALSE;
                    clear();
                    clean_cmd();
                    usr_inforREAD(temp_usr->id);
                    clear();
                    clean_cmd();
                    if (reset==TRUE) printf("您的信息已重置!\n");
                    reset=FALSE;
                    if (pwd_reset==TRUE) printf("您的密码已更改!\n");
                    pwd_reset=FALSE;
                    study_page=TRUE;
                    break;
                }
                case 2://课程信息
                {
                    clear();
                    subject_print();//打印信计课程信息
                    break;
                }
                case 3://修改用户的信息
                {
                    clear();
                    study_page=FALSE;
                    printf("请再次输入您的密码以确认:\n");
                    printf("Password:");
                    scanf("%d",&usr_pwd);
                    clear();
                    clean_cmd();
                    if(usr_pwd==temp_usr->password) printf("权限正常\n");
                    else 
                    {
                        clear();
                        printf("密码错误 权限不够\n");
                        study_page=TRUE;
                        break;
                    }
                    clean_cmd();
                    usr_inforWRITE(temp_usr->id);
                    study_page=TRUE;
                    break;
                }
                case 4://学习记录
                {
                    clear();
                    study_page=FALSE;
                    record=TRUE;
                    study_record(temp_usr->id);
                    study_page=TRUE;
                    record=FALSE;
                    break;
                }
                case 5://返回主界面但不注销
                {
                    study_page=FALSE;
                    firstpage=TRUE;
                    offline=FALSE;
                    clear();
                    break;
                }
                case 6://注销并退出
                {
                    study_page=FALSE;
                    offline=TRUE;
                    temp_usr=NULL;
                    clear();
                    firstpage=TRUE;
                    printf("您已注销.\n");
                    break;
                }
                default:
                {
                    clean_cmd();
                    clear();
                    printf("studyMode:Invalid command!\n");
                    break;
                }
            }
        }
        if(cmd==5||cmd==6) break;
    }
}
void write_grades(long newgrades,int id)//记录学习
{
    char filename[30]="",txt[7]=".txt";
    char data;
    int i,j,sizeg,n=1,sizes;
    strcpy(filename,All_usr[id].name);
    strcat(filename,txt);
    info_record=fopen(filename,"w");
    sizeg=num_size(newgrades);
    if(tp_record[1]==0)
    {
        fputc('r',info_record);
        fputc('1',info_record);
        fputc(':',info_record);
        for(i=1;i<=sizeg;i++)
        {
            fputc(num_ascii(num_getn(newgrades,i)),info_record);
        }
        fputc('!',info_record);
        fclose(info_record);
        tp_record[1]=newgrades;
        return;
    }
    else
    {
        while(tp_record[n+1]!=0) ++n;
    }
    tp_record[n+1]=newgrades;
    for(i=1;i<=n+1;i++)
    {
        fputc('r',info_record);
        sizes=num_size(i);
        if(sizes==1) fputc(num_ascii(i),info_record);
        else
        {
            fputc(num_ascii(i/10),info_record);
            fputc(num_ascii(i%10),info_record);
        }
        fputc(':',info_record);
        sizeg=num_size(tp_record[i]);
        for(j=1;j<=sizeg;j++)
        {
            fputc(num_ascii(num_getn(tp_record[i],j)),info_record);
        }
        if(i==n+1) fputc('!',info_record);
        else fputc('\n',info_record);
    }
    fclose(info_record);
}
void study_record(usrid)//读取ID用户记录
{
    char filename[30]="",txt[7]=".txt";
    int i=1,j=1,n=1,times=1,s;
    char snum[10]="",gnum[10],data='a';
    long sn=0,gn=0;
    strcpy(filename,All_usr[usrid].name);//拷贝名字
    strcat(filename,txt);//追加后缀名
    info_record=fopen(filename,"r");//r(id):grades
    while(data!='!')
    {
        data=fgetc(info_record);//读第一个字符
        if(data=='r')//有效分数开头
        {
            data=fgetc(info_record);//序数第一个字符
            if(chnu_to_innu(data)==0)
            {
                clear();
                fclose(info_record);
                printf("您还没有学习记录!\n");
                printf("开始学习吧!Go Go Go!!!\n");
                tp_record[1]=0;
                return;
            }
            snum[1]=data;
            times=1;
            sn=0;
            gn=0;
            while(data!=':')
            {
                data=fgetc(info_record);
                if(data!=':')
                {
                    ++times;
                    snum[times]=data;
                }
            }//data=':'
            for(i=1;i<=times;i++)
            {
                sn+=chnu_to_innu(snum[i])*pow(10,times-i);//序号
            }
            data=fgetc(info_record);//分数第一个字符
            gnum[1]=data;
            times=1;
            while(data!='\n'&&data!='!')
            {
                data=fgetc(info_record);
                if(data!='\n'&&data!='!')
                {
                    ++times;
                    gnum[times]=data;
                }
            }//data='\n' or data='!'
            for(i=1;i<=times;i++)
            {
                gn+=chnu_to_innu(gnum[i])*pow(10,times-i);//记录
            }
            tp_record[sn]=gn;
            if(data!='!') ++n;
        }
    }
//    tp_record[n+1]='!';
    fclose(info_record);
    if(record==FALSE) return;
    printf("您的最近的记录情况(最多记录14条):\n");
    printf("--------------------------------------------\n");
    if(n<=14)
    {
        for(i=1;i<=n;i++)
        {
            s=tp_record[i]/3;
            printf("[%d]:",i);
            for(j=1;j<=s;j++) printf("||");
            printf("\n");
        }
    }
    else
    {
        for(i=n-14;i<=n;i++)
        {
            s=tp_record[i]/3;
            printf("[%d]:",i);
            for(j=1;j<=s;j++) printf("||");
            printf("\n");
        }
    }
    printf("--------------------------------------------\n");
    printf("(注:未显示的行说明分数过低<3)\n");
    printf("Press Enter to continue...");
    getchar();
    clear();
}
void subject_print()
{
    printf("-----------------------------------------------\n");
    printf("|                 <大一上>                    |\n");
    printf("-----------------------------------------------\n");
    printf("|数学分析1 高等代数1 解析几何 计算机导论      |\n");
    printf("-----------------------------------------------\n");
    printf("|                 <大一下>                    |\n");
    printf("-----------------------------------------------\n");
    printf("|数学分析2 高等代数2 大学物理(二) c语言       |\n");
    printf("-----------------------------------------------\n");
    printf("|                 <大二上>                    |\n");
    printf("-----------------------------------------------\n");
    printf("|数学分析3 常微分方程 物理实验(二) 运筹学     |\n");
    printf("|MATLAB程序设计 离散数学 数据结构             |\n");
    printf("-----------------------------------------------\n");
    printf("|                 <大二下>                    |\n");
    printf("-----------------------------------------------\n");
    printf("|概率论与数理统计 c++程序设计 信息与密码学    |\n");
    printf("|数学建模 计算机组成原理 数值计算             |\n");
    printf("-----------------------------------------------\n");
    printf("|                 <大三上>                    |\n");
    printf("-----------------------------------------------\n");
    printf("|软件工程 数据库原理 数字信号处理 *代数编码   |\n");
    printf("-----------------------------------------------\n");    
    printf("|                 <大三下>                    |\n");
    printf("-----------------------------------------------\n");
    printf("|微分方程数值解 *算法分析与设计 计算机操作系统|\n");
    printf("|计算机网络技术 *数字逻辑基础 *信息管理系统   |\n");
    printf("|编译原理 *计算数论 *复变函数论 汇编语言      |\n");
    printf("-----------------------------------------------\n");
    printf("|                 <大四上>                    |\n");
    printf("-----------------------------------------------\n");
    printf("|计算机新技术 数据挖掘 人工智能 计算机图形学  |\n");
    printf("|数据存储与恢复技术 *泛函分析 *软件开发环境   |\n");
    printf("|网路编程基础 linux操作系统实践 *专业英语     |\n");
    printf("|数学物理方程 实变函数论                      |\n");
    printf("-----------------------------------------------\n");
    printf("|              <大四下(毕业)>                 |\n");
    printf("-----------------------------------------------\n");
    printf("|社会实践 专业实习 毕业论文设计               |\n");
    printf("-----------------------------------------------\n");
    printf("Press Enter to continue...");
    getchar();
    clear();
}
void put_data(int usrid)//导入用户分数数据
{
    int grades_length=1,id_length=1;
    char data,str[10]="";
    int id=0,i=0,lv=1;
    long final=0;
    str[0]='a';//将每个数字以字符的形式存入该数组
    usr_file=fopen(".usr_data.txt","r");//读取用户的信息
    for(i=0;i<100;i++)
    {
        All_usr[usrid].infor.grade=0;
    }
    while(1)
    {
        data=fgetc(usr_file);
        if(data=='i')//判断数据起始位置
        {
            data=fgetc(usr_file);
            id=chnu_to_innu(data);
            id_length=1;
            while(data!=':')//计算id长度
            {
                data=fgetc(usr_file);
                if(data!=':') 
                {
                    id=id*10+chnu_to_innu(data);
                    id_length=2;
                }
            }//读到冒号就退出
            if(id!=usrid)
            {
                continue;
            }
            data=fgetc(usr_file);//开始读取学习分数
            str[1]=data;
            grades_length=1;
            if(data=='0') return;
            while(data!='!')//计算分数长度
            {
                data=fgetc(usr_file);
                if(data!='!') 
                {
                    ++grades_length;
                    str[grades_length]=data;
                }
            }
            for(i=0;i<grades_length;i++)//将字符数据转换为数字数据
            {
                final=final+chnu_to_innu(str[grades_length-i])*pow(10,i);
            }
            All_usr[usrid].infor.grade=final;//导入
            while(1)
            {
                if(final>=rulers_function(lv)&&final<rulers_function(lv+1))
                {
                    All_usr[usrid].infor.level=lv;
                    break;
                }
                else ++lv;
            }
            fclose(usr_file);//关闭数据文件
            return;
        }
    }
}
int num_size(long num)//统计数字长度
{
    int size=0;
    if(num/10==0) return 1;
    while(num!=0)
    {
        num=num/10;
        ++size;
    }
    return size;
}
int num_ascii(int num)//获取数字的ASCII码
{
    return 48+num;//0的ASCII码为48
}
int chnu_to_innu(char num)//将字符转换为数字
{
    switch(num)
    {
        case '0':return 0;
        case '1':return 1;
        case '2':return 2;
        case '3':return 3;
        case '4':return 4;
        case '5':return 5;
        case '6':return 6;
        case '7':return 7;
        case '8':return 8;
        case '9':return 9;
    }
    return 0;
}
int num_getn(long num,int n)//第n个数字提取
{
    int size;
    long a,b;
    size=num_size(num);
    if(size==1&&n==1) return num%10;
//    if(n==1) return num%10;
    else
    {
        a=num%(long)pow(10,size-n);
        b=num%(long)pow(10,size-n+1);
        return (b-a)/(long)pow(10,size-n);
    }
}
long rulers_function(int x)//量化函数
{
    if (x==1) return 0;
    else if(x==2) return 40;
    else return b(x-2)+a(x-2)+rulers_function(x-1); 
}
void usrscreen(int usrid)
{
    printf("------------------------------\n");
    printf("|◆    >您的学习信息资料<    ◆|\n");
    printf("|----------------------------|\n");
    printf("|        1.个人信息          |\n");
    printf("|        2.课程信息          |\n");
    printf("|        3.信息管理*         |\n");
    printf("|        4.信息分析          |\n");
    printf("|        5.返回              |\n");
    printf("|        6.注销              |\n");
    printf("|◆                          ◆|\n");
    printf("------------------------------\n");
    printf("|当前用户： ●%s●  |\n",All_usr[usrid].name);
    printf("Please enter your choice(1-6):");
}
void usr_inforREAD(int usrid)
{
    char cmd,filename[30]="",txt[7]=".txt";
    int newpwd,temp;
    ranks usr_rank;
    printf("|         %s的资料卡            \n",All_usr[usrid].name);
    printf("|----------------------------|\n");
    printf("|ID:%d\n",All_usr[usrid].id);
    printf("|Lv. ★%d★   Rank:",All_usr[usrid].infor.level);
    input(&usr_rank,All_usr[usrid].infor.level);
    output(usr_rank);
    printf("\n");
    printf("|Grades: %ld  (Next Level:%ld)\n",All_usr[usrid].infor.grade,rulers_function(All_usr[usrid].infor.level+1));
    printf("|----------------------------|\n");
    printf("|o键回车进行重置\n");
    printf("|p键回车修改密码\n");
    printf("|or press Enter to exit:");
    cmd=getchar();
    if (cmd=='p')
    {
        while(1)
        {
            printf("---------------------------\n");
            printf("请输入您的新密码:");
            scanf("%d",&newpwd);
            temp=newpwd;
            printf("再次输入您的密码:");
            scanf("%d",&newpwd);
            if(newpwd==temp) break;
            printf("密码不一致!!!\n");
        }
        All_usr[usrid].password=newpwd;
        newer=FALSE;
        regist();
        pwd_reset=TRUE;
        return;
    }
    else if (cmd=='o')
    {
        All_usr[usrid].infor.level=1;
        All_usr[usrid].infor.grade=0;
        //
        strcpy(filename,All_usr[usrid].name);
        strcat(filename,txt);
        newrecord=fopen(filename,"w");
        fputc('r',newrecord);
        fputc('0',newrecord);
        fputc(':',newrecord);
        fputc('0',newrecord);
        fputc('!',newrecord);
        fclose(newrecord);
        write_data(usrid);//保存
        reset=TRUE;
        return ;
    }
    else if(cmd=='\n')
    {
        clear();
        clean_cmd();
        return;
    }
}
void usr_inforWRITE(int usrid)//修改用户的信息
{
    char cmd;
    ranks usr_rank;
    long new_grades;
    printf("|         %s的资料卡          \n",All_usr[usrid].name);
    printf("|----------------------------|\n");
    printf("|Id:%d\n",All_usr[usrid].id);
    printf("|Lv. ★%d★   Rank:",All_usr[usrid].infor.level);
    input(&usr_rank,All_usr[usrid].infor.level);
    output(usr_rank);
    printf("\n");
    printf("|Grades: %ld  (Gext Level:%ld)\n",All_usr[usrid].infor.grade,rulers_function(All_usr[usrid].infor.level+1));
    printf("|----------------------------|\n");
    printf("Press Enter to change...");
    cmd=getchar();
    clear();
    w_able=TRUE;
    new_grades=study_rulers();
    write_grades(new_grades,usrid);//将新分数写入记录
    All_usr[usrid].infor.grade+=new_grades;//study_rulers();//加入新得到的分数
    new_grades=0;
//    clean_cmd();
    if(All_usr[usrid].infor.grade>=rulers_function(All_usr[usrid].infor.level+1)) 
    {
        ++All_usr[usrid].infor.level;
        lv_up=TRUE;
    }
    write_data(usrid);//将修改后的数据保存入文件
    getchar();
    if (cmd=='q') 
    clear();
    clean_cmd();
    if(lv_up==TRUE) printf("♬ ♪ ♬ ♪祝贺您，您升级了！^_^\n");
    lv_up=FALSE;
    return;
}
void write_data(int usrid)//将数据写入文件
{
    int id=0,i=0;
    int size_grades=0,size_id=0;
    usr_file=fopen(".usr_data.txt","w");
    for(id=0;id<100;id++)
    {
        size_grades=num_size(All_usr[id].infor.grade);//计算分数长度
        size_id=num_size(id);//计算id长度
        fputc('i',usr_file);//有效数据段开头
        if(size_id==1) fputc(num_ascii(id),usr_file);
        else
        {
            for(i=1;i<=size_id;i++) fputc(num_ascii(num_getn(id,i)),usr_file);//id字符导入
        }
        fputc(':',usr_file);//有效分数开头
        if(size_grades==1) fputc(num_ascii(All_usr[id].infor.grade),usr_file);
        else
        {
            for(i=1;i<=size_grades;i++)
            {
                fputc(num_ascii(num_getn(All_usr[id].infor.grade,i)),usr_file);//分数字符导入
            }
        }
        fputc('!',usr_file);//结尾字符'!'导入
        fputc('\n',usr_file);//数据换行
    }
    fclose(usr_file);
}
int input(ranks *r,int lv)
{
    switch(lv)
    {
        case 1:*r=xiaobai;return 0;
        case 2:*r=xiaobai;return 0;
        case 3:*r=mengxin;return 0;
        case 4:*r=mengxin;return 0;
        case 5:*r=xuetu;return 0;
        case 6:*r=xuetu;return 0;
        case 7:*r=zhizhe;return 0;
        case 8:*r=zhizhe;return 0;
        case 9:*r=dashi;return 0;
        case 10:*r=dashi;return 0;       
        case 11:*r=shuoshi;return 0;
        case 12:*r=shuoshi;return 0;       
        case 13:*r=boshi;return 0;
        case 14:*r=boshi;return 0;       
        case 15:*r=yanjiusheng;return 0;
        case 16:*r=yanjiusheng;return 0;      
        case 17:*r=xuezhe;return 0;
        case 18:*r=xuezhe;return 0;       
        case 19:*r=jiaoshou;return 0;
        case 20:*r=jiaoshou;return 0;      
        case 21:*r=xuexijia;return 0;
        case 22:*r=xuexijia;return 0;       
        case 23:*r=zsxx;return 0;
        default:*r=zsxx;return 0;

    }
    return 0;
}
void output(ranks r)
{
    switch(r)
    {
        case xiaobai:printf("小白");return;
        case mengxin:printf("萌新");return;
        case xuetu:printf("学徒");return;
        case zhizhe:printf("智者");return;
        case dashi:printf("大师");return;
        case shuoshi:printf("硕士");return;
        case boshi:printf("博士");return;
        case yanjiusheng:printf("研究生");return;
        case xuezhe:printf("学者");return;
        case jiaoshou:printf("教授");return;
        case xuexijia:printf("学习家");return;
        case zsxx:printf("♚终生学习♚");return;
    }
}
long study_rulers()//量化规则修改或查看
{
    int numbers=0,database,new_grades=0;
    printf("-----------------------------\n");
    printf("|   这是学习考核标准:       |\n");
    printf("| 1.专业学习: 4分/小时      |\n");
    printf("| 2.编程: 3分/练习 6分/测试 |\n"); 
    printf("|         10分/工程         |\n");
    printf("| 3.单词记忆: 2分/20个      |\n");
    printf("| 4.听力练习: 1分/10分钟    |\n");
    printf("| 5.口语练习: 1分/10分钟    |\n");
    printf("| 6.解决问题: 1分/个        |\n");
    printf("-----------------------------\n");
    if(w_able==TRUE)
    {
        printf("请以(序号 数据项)的方式输入:\n");
        while(numbers!=6)
        {
            scanf("%d %d",&numbers,&database);
            new_grades=new_grades+rulers_date(numbers,database);
        }
        clear();
//        clean_cmd();
        printf("数据已更新!   已保存!\n");
        if(lv_up==TRUE) printf("祝贺您！您升级了！^_^\n");
        lv_up=FALSE;
        return new_grades;
    }
    return 0;
}
int rulers_date(int n,int data)
{
    switch(n)
    {
        case 1:
        {
            return 4*data;
        }
        case 2:
        {
            switch(data)
            {
                case 1: return 3;
                case 2: return 6;
                case 3: return 10;
            }
        }
        case 3:
        {
            return (data/20)*2;
        }
        case 4:
        {
            return (data/10);
        }
        case 5:
        {
            return (data/10);
        }
        case 6:
        {
            return data;
        }
    }
    return -1;
}
//######################################################
void help()
{
    char data;
    helptxt=fopen("tool_help.txt","r");
    data=fgetc(helptxt);
    while(!feof(helptxt))
    {
        printf("%c",data);
        data=fgetc(helptxt);
    }
    fclose(helptxt);
    printf("-------------------------------------------\n");
    printf("制作人:Skecis AI    制作时间:2017/10~11\n");
    printf("-------------------------------------------\n");
    printf("Press Enter to continue...");
    getchar();
    clear();
}
//######################################################
void usr_list()
{
    int i=0,n=0,admin_p=0;
    printf("请输入管理员密码:\n");
    printf("Password:");
    scanf("%d",&admin_p);
    getchar();
    if(admin_p!=All_usr[0].password)
    {
        clean_cmd();
        clear();
        printf("管理员密码错误!!\n");
        return;
    }
    clear();
    printf("            用户列表\n");
    printf("------------------------------\n");
    printf("ID  Name  Password\n");
    printf("------------------------------\n");
    for(i=0;i<100;i++)
    {
        if(All_usr[i].password==0) continue;
        printf("%d  %s  %d\n",i,All_usr[i].name,All_usr[i].password);
        ++n;
    }
    printf("------------------------------\n");
    printf("一共有%d名用户\n",n);
    printf("Press Enter to continue...");
    getchar();
    clear();
}
//######################################################
void regist()//账号注册函数
{
    int usr_id,usr_pwd,temp,newpwd;//新用户数据
    int i=0,j=0;
    char usr_name[10],ctemp='a',filename[30]="",txt[7]=".txt";
    char all_name[100][10];//100个名字
    usr_pwd_file=fopen(".usr_pwd.txt","w");//修改密码文件
    if(newer==TRUE)//注册模式
    {
        printf("-----------------------------\n");
        printf("|          用户注册         |\n");
        printf("-----------------------------\n");
        printf("|请输入您的ID:");
        scanf("%d",&usr_id);
        if(All_usr[usr_id].password!=0)//判断用户已经存在
        {
            clear();
            fclose(usr_pwd_file);
            printf("该ID账号已注册! 请更换!\n");
            regist();
            return;
        }
        printf("|新的用户名:");
        clean_cmd();
        scanf("%s",usr_name);
        usr_name[strlen(usr_name)]='!';//名字结尾处!
        getchar();
        printf("*****************************\n");
        clean_cmd();
        while(1)
        {
            printf("|输入您要使用的密码:");
            scanf("%d",&usr_pwd);
            temp=usr_pwd;
            printf("|再次输入确认您的密码:");
            scanf("%d",&usr_pwd);
            if(usr_pwd==temp) break;
            else printf("|您两次输入的密码不一致!!!\n");
            printf("-----------------------------\n");
            printf("|请重新输入:\n");
        }
        All_usr[usr_id].password=usr_pwd;
        clean_cmd();
        i=0;
        while(ctemp!='!')
        {
            ctemp=usr_name[i];
            if(ctemp!='!')
            {
                All_usr[usr_id].name[i]=ctemp;
                ++i;
            }
        }
//        strcpy(filename,All_usr[usr_id].name);
//        strcat(filename,txt);
//        newrecord=fopen(filename,"w");
//        fputc('r',newrecord);
//        fputc('0',newrecord);
//        fputc(':',newrecord);
//        fputc('0',newrecord);
//        fputc('!',newrecord);
//        fclose(newrecord);
    }//注册模式结尾
    //                      写密码文件
    for(i=0;i<100;i++)
    {
        for(j=0;j<strlen(All_usr[i].name);j++)
        {
            all_name[i][j]=All_usr[i].name[j];
        }
        all_name[i][strlen(All_usr[i].name)]='!';
    }
//    All_usr[usr_id].password=usr_pwd;
    for(i=0;i<100;i++)
    {
        j=0;
        fputc('$',usr_pwd_file);//数据条开始位置
        if(num_size(i)==1) fputc(num_ascii(i),usr_pwd_file);//写id
        else
        {
            fputc(num_ascii(i/10),usr_pwd_file);
            fputc(num_ascii(i%10),usr_pwd_file);
        }
        fputc(':',usr_pwd_file);
        fputc('#',usr_pwd_file);//名字开始位置
        if(All_usr[i].password==0)//用户是否存在
        {
            fputc('#',usr_pwd_file);
            fputc('!',usr_pwd_file);
            fputc('\n',usr_pwd_file);
            continue;
        }
        while(all_name[i][j]!='!')//写入名字
        {
            fputc(all_name[i][j],usr_pwd_file);
            ++j;
        }
        fputc('#',usr_pwd_file);
        for(j=0;j<3;j++)//写入密码
        {
            fputc(num_ascii(num_getn(All_usr[i].password,j+1)),usr_pwd_file);
        }
        fputc('!',usr_pwd_file);
        fputc('\n',usr_pwd_file);//数据段结尾
    }
    fclose(usr_pwd_file);
    clear();
    clean_cmd();
    if(newer==TRUE)//注册模式
    {
        strcpy(filename,All_usr[usr_id].name);
        strcat(filename,txt);
        newrecord=fopen(filename,"w");
        fputc('r',newrecord);
        fputc('0',newrecord);
        fputc(':',newrecord);
        fputc('0',newrecord);
        fputc('!',newrecord);
        fclose(newrecord);
        firstpage=TRUE;
        newer=FALSE;
        printf("新用户");
        for(j=0;j<strlen(All_usr[usr_id].name);j++) printf("%c",All_usr[usr_id].name[j]);
        printf("已添加!\n");
        return;
    }
}
