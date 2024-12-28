###### UNIX LAB ######


4)emulate the UNIX ls –l command
#include <stdio.h>
#include <conio.h>
#include <dos.h>
#include <dir.h>
#include <time.h>
void display_file_info(struct ffblk file);
void main()
{
struct ffblk file;
int done;
clrscr();
done = findfirst("*.*", &file, FA_ARCH | FA_RDONLY | FA_HIDDEN | FA_DIREC);
printf("Attributes    Size    Date      Time     Name\n");
printf("------------------------------------------------------\n");
while (!done) 
{
display_file_info(file);
done = findnext(&file);
}
getch();
}
void display_file_info(struct ffblk file)
{
struct tm *time_info;
unsigned year, month, day, hour, minute, second;
printf("%c", (file.ff_attrib & FA_RDONLY) ? 'r' : '-');
printf("%c", (file.ff_attrib & FA_HIDDEN) ? 'h' : '-');
printf("%c", (file.ff_attrib & FA_SYSTEM) ? 's' : '-');
printf("%c", (file.ff_attrib & FA_DIREC) ? 'd' : '-');
printf("%c", (file.ff_attrib & FA_ARCH) ? 'a' : '-');
printf("    ");
printf("%8lu", file.ff_fsize);
year = ((file.ff_fdate >> 9) & 0x7F) + 1980;
month = (file.ff_fdate >> 5) & 0x0F;
day = file.ff_fdate & 0x1F;
hour = (file.ff_ftime >> 11) & 0x1F;
minute = (file.ff_ftime >> 5) & 0x3F;
second = (file.ff_ftime & 0x1F) * 2;
printf("  %02d-%02d-%04d", day, month, year);
printf("  %02d:%02d:%02d", hour, minute, second);
printf("  %s\n", file.ff_name);
}


5)how to execute two commands concurrently with a command pipe. Ex: - ls –l | sort
#include<stdio.h>
#include<stdlib.h>
void main()
{
clrscr();
system("dir | sort");
//system("ls -l | sort");
getch();
}


6)Multiprogramming-Memory management-Implementation of fork (), wait (), exec() and exit (), System calls
#include <stdio.h>
#include <stdlib.h>
#include <dos.h>
void childProcess() {
printf("Child process.\n");
delay(1000);
}
void parentProcess() {
printf("Parent process.\n");
childProcess();
}
void executeCommand(const char *command) {
printf("Command: %s\n", command);
system(command);
}
void main() {
clrscr();
printf("Simulating fork(), wait(), exec(), exit() in DOS.\n");
parentProcess();
executeCommand("dir");
printf("Exiting program.\n");
exit(0);
getch();
}


###### OS LAB ######


1a)FCFS
#include<stdio.h>
void findavgTime(int bt[],int n)
{
int wt[10];
int tat[10];
int total_wt=0,total_tat=0;
int i;
wt[0]=0;
for(i=1;i<n;i++)
wt[i]=bt[i-1]+wt[i-1];
for(i=0;i<n;i++)
tat[i]=bt[i]+wt[i];
printf("Processes    Burst time    Waiting time      Turn around time\n");
for(i=0;i<n;i++)
{
total_wt+=wt[i];
total_tat+=tat[i];
printf("   %d            %d             %d                 %d\n",i+1,bt[i],wt[i],tat[i]);
}
printf("Avg. waiting time = %.2f\n",(float)total_wt/n);
printf("Avg. turn around time = %.2f\n",(float)total_tat/n);
}
void main()
{
int burst_time[]={4,13,6};
int n=sizeof burst_time/sizeof burst_time[0];
clrscr();
findavgTime(burst_time,n);
getch();
}


1b)SJF
//non-preemptive
#include<stdio.h>
void main()
{
int A[100][4],n,total=0;
int i,j,k,index;
float avg_wt,avg_tat;
clrscr();
printf("Enter no. of processes: ");
scanf("%d",&n);
printf("Enter burst time:\n");
for(i=0;i<n;i++)
{
printf("P%d: ",i+1);
scanf("%d",&A[i][1]);
A[i][0]=i+1;
}
for(i=0;i<n;i++)
{
index=i;
for(j=i+1;j<n;j++)
if(A[j][1] < A[index][1])
index=j;
for(k=0;k<2;k++)
{
int temp=A[i][k];
A[i][k] = A[index][k];
A[index][k] = temp;
}
}
A[0][2]=0;
for(i=1;i<n;i++)
{
A[i][2]=A[i-1][2] + A[i-1][1];
total += A[i][2];
}
avg_wt = (float)total/n;
total=0;
printf("P\tBT\tWT\tTAT\n");
for(i=0;i<n;i++)
{
A[i][3]=A[i][1]+A[i][2];
total += A[i][3];
printf("P%d\t%d\t%d\t%d\n",A[i][0],A[i][1],A[i][2],A[i][3]);
}
avg_tat = (float)total/n;
printf("Avg WT= %.2f\nAvg TAT= %.2f\n",avg_wt,avg_tat);
getch();
}
//preemptive
#include<stdio.h>
#include<limits.h>
void main()
{
int pid[]={1,2,3,4,5}, bt[]={3,2,9,10,4}, art[]={6,1,4,8,2};
int n=sizeof(pid)/sizeof(pid[0]),wt[10],tat[10],rt[10];
int i,j,t,complete;
int total_wt=0,total_tat=0;
clrscr();
for(i=0;i<n;i++)
rt[i]=bt[i];
for(complete=0,t=0;complete<n;t++)
{
int minm = INT_MAX, check=0,shortest;
for(j=0;j<n;j++)
{
if(art[j] <= t && rt[j] < minm && rt[j] > 0)
{
minm = rt[j];
shortest = j;
check=1;
}
}
if(!check) continue;
if(--rt[shortest]==0)
{
wt[shortest]=t+1-bt[shortest]-art[shortest];
if(wt[shortest]<0)
wt[shortest]=0;
complete++;
}
}
for(i=0;i<n;i++)
tat[i]=bt[i]+wt[i];
printf("P\tBT\tWT\tTAT\n");
for(i=0;i<n;i++)
{
total_wt += wt[i];
total_tat += tat[i];
printf("%d\t%d\t%d\t%d\n",pid[i],bt[i],wt[i],tat[i]);
}
printf("Avg WT = %.2f\nAvg TAT=%.2f\n",(float)total_wt/n,(float)total_tat/n);
getch();
}


1c)Priority
//Non-preemptive
#include<stdio.h>
#include<conio.h>
void main()
{
int i,j,n,tat[10],wt[10],bt[10],pid[10],pr[10],t,twt=0,ttat=0;
float awt,atat;
clrscr();
printf("Enter the no. of processes:");
scanf("%d",&n);
for(i=0;i<n;i++)
{
pid[i]=i;
printf("Enter the Burst Time of Process-%d:",i+1);
scanf("%d",&bt[i]);
printf("Enter the Priority of Process-%d:",i+1);
scanf("%d",&pr[i]);
}
for(i=0;i<n;i++)
for(j=i+1;j<n;j++)
{
if(pr[i]>pr[j])
{
t=pr[i];
pr[i]=pr[j];
pr[j]=t;
t=bt[i];
bt[i]=bt[j];
bt[j]=t;
t=pid[i];
pid[i]=pid[j];
pid[j]=t;
}
}
tat[0]=bt[0];
wt[0]=0;
for(i=1;i<n;i++)
{
wt[i]=wt[i-1]+bt[i-1];
tat[i]=wt[i]+bt[i];
}
printf("\n");
printf("P_ID\tPRIORITY\tBURST_TIME\tWAITING_TIME\tTURN_AROUND_TIME\n");
for(i=0;i<n;i++)
{
printf("\n%d\t%d\t\t%d\t\t%d\t\t%d",((pid[i])+1),pr[i],bt[i],wt[i],tat[i]);
}
for(i=0;i<n;i++)
{
ttat=ttat+tat[i];
twt=twt+wt[i];
}
awt=(float)twt/n;
atat=(float)ttat/n;
printf("\n\nAvg Waiting Time : %f \nAvg Turn Around Time : %f \n",awt,atat);
getch();
}
//Preemptive
#include<stdio.h>
void main()
{
int n, i, j;
int p[30],bt[30],pr[30],at[30],wt[30],tat[30],temp;
float avgWT = 0, avgTAT = 0;
clrscr();
printf("Enter number of processes: ");
scanf("%d", &n);
for (i = 0; i < n; i++) {
p[i] = i + 1;
printf("Enter BT, PR, AT for P%d: \n", p[i]);
scanf("%d %d %d", &bt[i], &pr[i], &at[i]);
}
for (i = 0; i < n - 1; i++)
for (j = i + 1; j < n; j++)
if (at[i] > at[j] || (at[i] == at[j] && pr[i] > pr[j])) {
temp = bt[i]; bt[i] = bt[j]; bt[j] = temp;
temp = pr[i]; pr[i] = pr[j]; pr[j] = temp;
temp = at[i]; at[i] = at[j]; at[j] = temp;
temp = p[i]; p[i] = p[j]; p[j] = temp;
}
wt[0] = 0;
for (i = 1; i < n; i++)
wt[i] = wt[i - 1] + bt[i - 1] - (at[i] - at[i - 1]);
for (i = 0; i < n; i++)
tat[i] = wt[i] + bt[i];
printf("\nPID\tBT\tPR\tAT\tWT\tTAT\n");
for (i = 0; i < n; i++)
printf("%d\t%d\t%d\t%d\t%d\t%d\n", p[i], bt[i], pr[i], at[i], wt[i], tat[i]);
for (i = 0; i < n; i++) {
avgWT += wt[i];
avgTAT += tat[i];
}
printf("\nAvg WT: %.2f", avgWT / n);
printf("\nAvg TAT: %.2f\n", avgTAT / n);
getch();
}


1d)Round robin
#include <stdio.h>
void findWaitingTime(int n, int bt[], int wt[], int quantum)
{
int i, rem_bt[40], t = 0, done;
for (i = 0; i < n; i++) rem_bt[i] = bt[i];
while (1){
done = 1;
for ( i = 0; i < n; i++){
if (rem_bt[i] > 0){
done = 0;
if (rem_bt[i] > quantum) t += quantum, rem_bt[i] -= quantum;
else t += rem_bt[i], wt[i] = t - bt[i], rem_bt[i] = 0;
}
}
if (done) break;
}
}
void findTurnAroundTime(int n, int bt[], int wt[], int tat[]){
int i;
for ( i = 0; i < n; i++) tat[i] = bt[i] + wt[i];}
void findavgTime(int n, int bt[], int quantum){
int i, wt[40], tat[40], total_wt = 0, total_tat = 0;
findWaitingTime(n, bt, wt, quantum);
findTurnAroundTime(n, bt, wt, tat);
printf("PN\tBT\tWT\tTAT\n");
for ( i = 0; i < n; i++)
printf("%d\t%d\t%d\t%d\n", i + 1, bt[i], wt[i], tat[i]), total_wt += wt[i], total_tat += tat[i];
printf("Average waiting time = %.2f\n", (float)total_wt / n);
printf("Average turn around time = %.2f\n", (float)total_tat / n);}
void main()
{
int burst_time[] = {10, 5, 8};
int n = sizeof(burst_time) / sizeof(burst_time[0]);
int quantum = 2;
clrscr();
findavgTime(n, burst_time, quantum);
getch();
}


2)Multiprogramming-Memory Management- Implementation of fork(), wait(), exec() and exit()
#include <stdio.h>
#include <stdlib.h>
#include <dos.h>
void childProcess() {
printf("Child process.\n");
delay(1000);
}
void parentProcess() {
printf("Parent process.\n");
childProcess();
}
void executeCommand(const char *command) {
printf("Command: %s\n", command);
system(command);
}
void main() {
clrscr();
printf("Simulating fork(), wait(), exec(), exit() in DOS.\n");
parentProcess();
executeCommand("dir");
printf("Exiting program.\n");
exit(0);
getch();
}


3a)MFT
#include<stdio.h> 
#include<conio.h> 
void main()
{
int ms,bs,nob,ef,n,mp[10],tif=0;
int i,p=0;
clrscr();
printf("Enter the total memory available (in Bytes) -- ");
scanf("%d",&ms);
printf("Enter the block size (in Bytes) -- ");
scanf("%d", &bs);
nob=ms/bs; ef=ms - nob*bs;
printf("\nEnter the number of processes -- ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter memory required for process %d (in Bytes)-- ",i+1);
scanf("%d",&mp[i]);
}
printf("\nNo. of Blocks available in memory -- %d",nob);
printf("\n\nPROCESS\tMEMORY REQUIRED\t ALLOCATED\tINTERNAL FRAGMENTATION");
for(i=0;i<n && p<nob;i++)
{
printf("\n %d\t\t%d",i+1,mp[i]);
if(mp[i] > bs)
printf("\t\tNO\t\t---");
else
{
printf("\t\tYES\t%d",bs-mp[i]);
tif = tif + bs-mp[i];
p++;
}
}
if(i<n)
printf("\nMemory is Full, Remaining Processes cannot be accomodated");
printf("\n\nTotal Internal Fragmentation is %d",tif);
printf("\nTotal External Fragmentation is %d",ef);
getch();
}


3b)MVT
#include<stdio.h> 
#include<conio.h> 
void main()
{
int ms,mp[10],i, temp,n=0; char ch = 'y';
clrscr();
printf("\nEnter the total memory available (in Bytes)-- ");
scanf("%d",&ms);
temp=ms; for(i=0;ch=='y';i++,n++)
{
printf("\nEnter memory required for process %d (in Bytes) -- ",i+1);
scanf("%d",&mp[i]);
if(mp[i]<=temp)
{
printf("\nMemory is allocated for Process %d ",i+1);
temp = temp - mp[i];
}
else
{
printf("\nMemory is Full"); break;
}
printf("\nDo you want to continue(y/n) -- ");
scanf(" %c", &ch);
}
printf("\n\nTotal Memory Available -- %d", ms);
printf("\n\n\tPROCESS\t\t MEMORY ALLOCATED ");
for(i=0;i<n;i++)
printf("\n \t%d\t\t%d",i+1,mp[i]);
printf("\n\nTotal Memory Allocated is %d",ms-temp);
printf("\nTotal External Fragmentation is %d",temp);
getch();
}


4)implement first fit, best fit and worst fit algorithm for memory management. 
#include <stdio.h>
#define BLOCKS 5
#define PROCESSES 3
void allocateMemory(int blocks[], int m, int processes[], int n, int choice) {
    int i, j, idx;
    for (i = 0; i < n; i++) {
        idx = -1;
        for (j = 0; j < m; j++) {
            if (blocks[j] >= processes[i] && (choice == 1 || (choice == 2 && (idx == -1 || blocks[j] < blocks[idx])) || (choice == 3 && (idx == -1 || blocks[j] > blocks[idx])))) {
                idx = j;
            }
	}
	if (idx != -1) {
	    printf("P%d allocated to B%d\n", i + 1, idx + 1);
	    blocks[idx] -= processes[i];
	} else {
	    printf("P%d cannot be allocated\n", i + 1);
	}
    }
}
void main() {
    int blocks[BLOCKS], processes[PROCESSES], i, blocks1[BLOCKS], blocks2[BLOCKS], blocks3[BLOCKS];
    clrscr();
    printf("Enter the sizes of %d blocks:\n", BLOCKS);
    for (i = 0; i < BLOCKS; i++) {
	printf("Block %d: ", i + 1);
	scanf("%d", &blocks[i]);
    }
    printf("Enter the sizes of %d processes:\n", PROCESSES);
    for (i = 0; i < PROCESSES; i++) {
	printf("Process %d: ", i + 1);
	scanf("%d", &processes[i]);
    }
    for (i = 0; i < BLOCKS; i++) {
	blocks1[i] = blocks[i];
	blocks2[i] = blocks[i];
	blocks3[i] = blocks[i];
    }
    printf("\nFirst Fit:\n");
    allocateMemory(blocks1, BLOCKS, processes, PROCESSES, 1);
    printf("\nBest Fit:\n");
    allocateMemory(blocks2, BLOCKS, processes, PROCESSES, 2);
    printf("\nWorst Fit:\n");
    allocateMemory(blocks3, BLOCKS, processes, PROCESSES, 3);
    getch();
}


5)Dead Lock Avoidance
#include<stdio.h>
#include<conio.h>
#include<string.h>
void main()
{
int alloc[10][10],max[10][10],avail[10],work[10],total[10],need[10][10];
int i,j,k,n,m,count=0,c=0;
char finish[10];
clrscr();
printf("Enter no. of processes and resources:\n");
scanf("%d%d",&n,&m);
for(i=0;i<=n;i++)
finish[i]='N';
printf("Enter claim matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&max[i][j]);
printf("Enter allocation matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&alloc[i][j]);
printf("Resource vector:\n");
for(i=0;i<m;i++)
scanf("%d",&total[i]);
for(i=0;i<m;i++)
avail[i]=0;
for(i=0;i<n;i++)
for(j=0;j<m;j++)
avail[j]+=alloc[i][j];
for(i=0;i<m;i++)
work[i]=avail[i];
for(j=0;j<m;j++)
work[j]=total[j]-work[j];
for(i=0;i<n;i++)
for(j=0;j<m;j++)
need[i][j]=max[i][j]-alloc[i][j];
A:for(i=0;i<n;i++)
{
c=0;
for(j=0;j<m;j++)
if((need[i][j]<=work[j])&&(finish[i]=='N'))
c++;
if(c==m)
{
printf("\nAll the resources can be allocated to Process %d",i+1);
printf("\nAvailable resources are:");
for(k=0;k<m;k++)
{
work[k]+=alloc[i][k];
printf("%4d",work[k]);
}
printf("\n");
finish[i]='Y';
printf("Process %d executed? : %c\n",i+1,finish[i]);
count++;
}
}
if(count!=n) goto A;
else
printf("\nSystem is in safe mode");
printf("\nGiven state is in safe state");
getch();
}


6)Dead Lock Prevention
#include<stdio.h>
#include<conio.h>
void main()
{
char job[10][10];
int time[10],tem[10],temp[10],safe[10];
int ind=1,avail,i,j,q,n,t;
clrscr();
printf("Enter no. of jobs: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter name and time: ");
scanf("%s%d",&job[i],&time[i]);
}
printf("Enter available resources: ");
scanf("%d",&avail);
for(i=0;i<n;i++)
{
temp[i]=time[i];
tem[i]=i;
}
for(i=0;i<n;i++)
for(j=i+1;j<n;j++)
{
if(temp[i]>temp[j])
{
t=temp[i];temp[i]=temp[j];temp[j]=t;
t=tem[i];tem[i]=tem[j];tem[j]=t;
}
}
for(i=0;i<n;i++)
{
q=tem[i];
if(time[q]<=avail)
{
safe[ind]=tem[i];
avail=avail-tem[q];
//printf("%s",job[safe[ind]]);
ind++;
}
else
{
printf("No safe sequence\n");
}
}
printf("Safe sequence is:\n");
for(i=1;i<ind;i++)
printf("%s %d\n",job[safe[i]],time[safe[i]]);
getch();
}


7a)FIFO
#include <stdio.h>
#include <stdlib.h>
#define MAX_FRAMES 3
void fifoPageReplacement(int pages[], int n, int capacity) {
int frames[MAX_FRAMES];
int pageFaults = 0;
int i, j, front = 0;
for ( i = 0; i < capacity; i++) { frames[i] = -1; }
for ( i = 0; i < n; i++) {
int currentPage = pages[i];
int pageFound = 0;
for ( j = 0; j < capacity; j++) {
if (frames[j] == currentPage) { pageFound = 1; break; }}
if (!pageFound) {
frames[front] = currentPage;
front = (front + 1) % capacity;
pageFaults++; }
printf("Page %d loaded into memory.\n", currentPage);
}
printf("\nTotal page faults: %d\n", pageFaults);
}
void main() {
int pages[] = {7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 0};
int n = sizeof(pages) / sizeof(pages[0]);
int capacity = 3;
clrscr();
fifoPageReplacement(pages, n, capacity);
getch();
}


7b)LRU
#include <stdio.h>
#define MAX_FRAMES 10
void simulateLRU(int pages[], int numPages, int numFrames) {
int frames[MAX_FRAMES] = {-1};
int pageFaults = 0, i, j, found, lruIndex;
for (i = 0; i < numPages; i++) {
found = 0;
for (j = 0; j < numFrames && !found; j++) { if (frames[j] == pages[i]) found = 1; }
if (!found) { pageFaults++;
for (j = 0; j < numFrames && frames[j] != -1; j++);
if (j < numFrames) frames[j] = pages[i];
else {
lruIndex = 0;
for (j = 1; j < numFrames; j++) {
if (frames[j] != pages[i]) lruIndex = j; }
frames[lruIndex] = pages[i]; }}
printf("Frames: ");
for (j = 0; j < numFrames; j++) {
if (frames[j] != -1) printf("%d ", frames[j]);
} printf("\n" ); }
printf("Page faults: %d\n", pageFaults);
}
void main() {
int pages[] = {7, 0, 1, 2, 0, 3, 0, 4, 2, 3, 0, 3, 2};
int numPages = sizeof(pages) / sizeof(pages[0]);
clrscr();
simulateLRU(pages, numPages, 3);
getch();
}


7c)LFU
#include <stdio.h>
#include <conio.h>
typedef struct { int page, freq, lastUsed; } Page;
int findLFU(Page f[], int n) { int i, idx = 0;
for (i = 1; i < n; i++) {
if (f[i].freq < f[idx].freq || (f[i].freq == f[idx].freq && f[i].lastUsed < f[idx].lastUsed)) {
idx = i; }} return idx; }
int lfu(int pages[], int n, int size) { Page f[26], temp;
int i, j, count = 0, faults = 0;
for (i = 0; i < n; i++) { int p = pages[i], found = 0;
for (j = 0; j < count; j++) {
if (f[j].page == p) { f[j].freq++; f[j].lastUsed = i;
found = 1; break; }}
if (!found) { faults++;
if (count < size) {
temp.page = p;
temp.freq = 1;
temp.lastUsed = i;
f[count++] = temp; } else { int idx = findLFU(f, size);
temp.page = p;
temp.freq = 1;
temp.lastUsed = i;
f[idx] = temp;
}}} return faults; }
void main() {
int pages[] = {1, 2, 3, 2, 1, 4, 2, 3};
clrscr();
printf("Page Faults: %d\n", lfu(pages, sizeof(pages) / sizeof(pages[0]), 3));
getch(); }


8a)Sequenced
#include<stdio.h> 
int main() 
{ 
int f[50], i, st, j, len, c, k; 
clrscr();
for(i = 0; i < 50; i++) f[i] = 0;
X:
printf("Enter starting block & length of file: ");
scanf("%d%d", &st, &len);
for(j = st; j < (st + len); j++)
if(f[j] == 0)
{
f[j] = 1;
printf("%d->%d\n", j, f[j]);
}
else
{
printf("Block already allocated");
break;
}
if(j == (st + len))
printf("File is allocated to disk");
printf("\nWant to enter more files? (y-1/n-0): ");
scanf("%d", &c);
if(c == 1)
goto X;
else
return 0;
}


8b)Indexed
#include<stdio.h>
int f[50],i,k,j,inde[50],n,c,count=0,p;
int main()
{
clrscr();
for(i=0;i<50;i++) f[i]=0;
x: printf("Enter index block: "); scanf("%d",&p);
if(f[p]==0)
{ f[p]=1;
printf("Enter no. of files on index: "); scanf("%d",&n);
}
else
{
printf("Block already allocated\n");
goto x;
}
for(i=0;i<n;i++) scanf("%d",&inde[i]); for(i=0;i<n;i++) if(f[inde[i]]==1)
{
printf("Block already allocated");
goto x;
}
for(j=0;j<n;j++) f[inde[j]]=1;
printf("Allocated"); printf("\nFile indexed"); for(k=0;k<n;k++)
printf("\n%d->%d:%d",p,inde[k],f[inde[k]]);
printf("\nEnter 1 to enter more files or 0 to exit: "); scanf("%d",&c);
if(c==1) goto x;
else
return 0;
}


8c)Linked
#include<stdio.h>
int main()  {
int f[50],p,i,j,k,a,st,len,n,c;
clrscr();
for(i=0;i<50;i++) f[i]=0;
printf("How many blocks are already allocated? : "); scanf("%d",&p);
printf("Enter block no's that are already allocated: ");
for(i=0;i<p;i++) {
scanf("%d",&a); f[a]=1;
}
X:
printf("Enter starting index block & length: "); scanf("%d%d",&st,&len); k=len;
for(j=st;j<(k+st);j++)
{
if(f[j]==0)
{ f[j]=1;
printf("%d->%d\n",j,f[j]);
}
else
{
printf("%d->File is already allocated\n",j);
k++;
}
}
printf("Want to enter one more file? (yes-1/no-0): ");
scanf("%d",&c); if(c==1)
goto X; else
return 0;
}
