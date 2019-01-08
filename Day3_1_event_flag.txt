#include"includes.h"
#include"system_stm32f10x.h"
#include"OS_cpu.h"

void task1(void *arg);
void task2(void *arg);
void task3(void *arg);
void task4(void *arg);
void task5(void *arg);
void task6(void *arg);


#define STACK_SIZE 100

OS_STK task_stk1[100];
OS_STK task_stk2[100];
OS_STK task_stk3[100];
OS_STK task_stk4[100];
OS_STK task_stk5[100];
OS_STK task_stk6[100];

OS_FLAG_GRP *Task_start;
OS_FLAGS flag;

#define flag_1 0x01
#define flag_2 0x02

INT8U a=0, i, j, ch, num;


int main(void)
{
	INT8U ret, err;
	OSInit();
	ret=OSTaskCreate(&task1,(void *)0,&task_stk1[100-1],1);
	if(ret!=OS_ERR_NONE)
		printf("task(producer 1) creation failed\n\r");

	ret=OSTaskCreate(&task2,(void *)0,&task_stk2[100-1],4);
	if(ret!=OS_ERR_NONE)
	{
		printf("task(producer 2) creation failed\n\r");
	}
	ret=OSTaskCreate(&task3,(void *)0,&task_stk3[100-1],5);
	if(ret!=OS_ERR_NONE)
	{
		printf("task(consumer) creation failed\n\r");
	}

	Task_start = OSFlagCreate(0x00, &err);

	OSStart();
    while(1)
    {
    	printf("hello..\n\r");
    }
}


void task1(void *arg)
{
	INT8U time, ret, err;

	for( ; ; )
	{
		OS_CPU_SysTickInit(SystemCoreClock/OS_TICKS_PER_SEC);
		printf("task 1\n\r");
		OSTimeDly(0.5*OS_TICKS_PER_SEC);
		flag = OSFlagPend(Task_start, flag_1+flag_2, OS_FLAG_WAIT_SET_ALL, 100, &err);

		if(flag)
		{
			time = OSTimeGet();
			printf("The timer after two flags are set %d\n\r", time);

			ret=OSTaskCreate(&task4,(void *)0,&task_stk4[100-1],4);
				if(ret!=OS_ERR_NONE)
				{
					printf("task4 creation failed\n\r");
				}
				ret=OSTaskCreate(&task5,(void *)0,&task_stk5[100-1],5);
				if(ret!=OS_ERR_NONE)
				{
					printf("task5 creation failed\n\r");
				}
				ret=OSTaskCreate(&task6,(void *)0,&task_stk6[100-1],6);
				if(ret!=OS_ERR_NONE)
				{
					printf("task6 creation failed\n\r");
				}
		    OSTaskDel(OS_PRIO_SELF);
		}
	}
}

void task2(void *arg)
{
	INT8U err;
	for( ; ; )
	{
		printf("task2 \n\r");

		for(ch=65; ch<=91; ch++)
		{
			for(i=0; i<5; i++)
			{
				if((ch+i) == 'Z')
				{
					printf("%c \t\r", ch+i);
					goto label;
				}
				else
					printf("%c \t\r", ch+i);
			}
			ch = ch+i-1;
			printf("\n\r");
			OSTimeDly(0.5*OS_TICKS_PER_SEC);
		}
label:
		flag = OSFlagPost(Task_start, flag_1, OS_FLAG_SET, &err);
		OSTaskDel(OS_PRIO_SELF);

	}
}

void task3(void *arg)
{
	INT8U err;
	for( ; ; )
	{
		printf("task3 \n\r");
		for(num=1; num<=50; num++)
		{
			for(i=0; i<10; i++)
			{
				printf("%d \t\r", num+i);
			}
			num = num+i-1;
			printf("\n\r");
			OSTimeDly(0.6*OS_TICKS_PER_SEC);
		}
		flag = OSFlagPost(Task_start, flag_2, OS_FLAG_SET, &err);
		OSTaskDel(OS_PRIO_SELF);
	}
}

void task4(void *arg)
{
	for(;;)
	{
		printf("In dynamic task1 \n\r");
		OSTimeDly(1*OS_TICKS_PER_SEC);
	}
}

void task5(void *arg)
{
	for(;;)
	{
		printf("In dynamic task2 \n\r");
		OSTimeDly(1*OS_TICKS_PER_SEC);
	}
}

void task6(void *arg)
{
	for(;;)
	{
		printf("In dynamic task3 \n\r");
		OSTimeDly(1*OS_TICKS_PER_SEC);
	}
}

