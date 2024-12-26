## impl in cpp
```cpp
#include <iostream>
#include <vector>
#include <array>
#include <algorithm>

constexpr int n = /* Number of processes */;
constexpr int m = /* Number of resources */;

struct State {
    std::array<int, m> resource;
    std::array<int, m> available;
    std::array<std::array<int, m>, n> claim;
    std::array<std::array<int, m>, n> alloc;
};

// Function to check if a process can be safely allocated
bool isSafe(const State& s) {
    std::array<int, m> currentAvail = s.available;
    std::vector<int> rest(n);
    for (int i = 0; i < n; ++i) {
        rest[i] = i;
    }

    bool possible = true;
    while (possible) {
        int foundIndex = -1;
        for (int i = 0; i < rest.size(); ++i) {
            int processIndex = rest[i];
            bool canExecute = true;
            for (int j = 0; j < m; ++j) {
                if (s.claim[processIndex][j] - s.alloc[processIndex][j] > currentAvail[j]) {
                    canExecute = false;
                    break;
                }
            }
            if (canExecute) {
                foundIndex = i;
                break;
            }
        }
        
        if (foundIndex != -1) {
            int processIndex = rest[foundIndex];
            for (int j = 0; j < m; ++j) {
                currentAvail[j] += s.alloc[processIndex][j];
            }
            rest.erase(rest.begin() + foundIndex);
        } else {
            possible = false;
        }
    }

    return rest.empty();
}

// Function to simulate resource allocation
bool allocateResources(State& s, int processIndex, const std::array<int, m>& request) {
    for (int j = 0; j < m; ++j) {
        if (s.alloc[processIndex][j] + request[j] > s.claim[processIndex][j]) {
            std::cout << "Error: Total request exceeds claim" << std::endl;
            return false;
        } else if (request[j] > s.available[j]) {
            std::cout << "Suspend process" << std::endl;
            return false;
        }
    }

    // Simulate allocation
    for (int j = 0; j < m; ++j) {
        s.alloc[processIndex][j] += request[j];
        s.available[j] -= request[j];
    }

    State newState = s;
    if (isSafe (newState)) {
        s = newState;
        return true;
    } else {
        // Restore original state
        for (int j = 0; j < m; ++j) {
            s.alloc[processIndex][j] -= request[j];
            s.available[j] += request[j];
        }
        std:: cout << "Suspend process" << std:: endl;
        return false;
    }
}

int main () {
    State state;
    // Initialize state's data members here

    int processIndex = /* Index of the requesting process */;
    std::array<int, m> request = /* Requested resources */;

    if (allocateResources (state, processIndex, request)) {
        std:: cout << "Resources allocated successfully" << std:: endl;
    }

    return 0;
}

```

## impl in C
```c
/*
Operating Systems
Banker's Algorithm
*/

#include <stdio.h>

int m, n, i, j, al[10][10], max[10][10], av[10], need[10][10], temp, z, y, p, k;

void main() {

  printf("\n Enter no of processes : ");
  scanf("%d", &m); // enter numbers of processes

  printf("\n Enter no of resources : ");
  scanf("%d", &n); // enter numbers of resources

  for (i = 0; i < m; i++) {
    for (j = 0; j < n; j++) {
      printf("\n Enter instances for al[%d][%d] = ", i,j); // al[][] matrix is for allocated instances
      scanf("%d", &al[i][j]);
      // al[i][j]=temp;
    }
  }

  for (i = 0; i < m; i++) {
    for (j = 0; j < n; j++) {
      printf("\n Enter instances for max[%d][%d] = ", i,j); // max[][] matrix is for max instances
      scanf("%d", &max[i][j]);
    }
  }

  for (i = 0; i < n; i++) {
    printf("\n Available Resource for av[%d] = ",i); // av[] matrix is for available instances
    scanf("%d", &av[i]);
  }

  // Print allocation values
  printf("Alocation Values :\n");
  for (i = 0; i < m; i++) {
    for (j = 0; j < n; j++) {
      printf(" \t %d", al[i][j]); // printing allocation matrix
    }
    printf("\n");
  }

  printf("\n\n");

  // Print max values
  printf("Max Values :\n");
  for (i = 0; i < m; i++) {
    for (j = 0; j < n; j++) {
      printf(" \t %d", max[i][j]); // printing max matrix
    }
    printf("\n");
  }

  printf("\n\n");

  // Print need values
  printf("Need Values :\n");
  for (i = 0; i < m; i++) {
    for (j = 0; j < n; j++) {
      need[i][j] = max[i][j] - al[i][j]; // calculating need matrix
      printf("\t %d", need[i][j]);       //  printing need matrix
    }
    printf("\n");
  }

  p = 1; // used for terminating while loop
  y = 0;
  while (p != 0) {

    for (i = 0; i < m; i++) {
      z = 0;
      for (j = 0; j < n; j++) {
        if (need[i][j] <= av[j] &&
            (need[i][0] != -1)) { // comparing need with available instance and
                                  // checking if the process is done
                                  // or not
          z++;                    // counter if condition TRUE
        }
      }

      if (z == n) { // if need<=available TRUE for all resources then condition
                    // is TRUE

        for (k = 0; k < n; k++) {
          av[k] += al[i][k]; // new work = work + allocated
        }
        printf("\n SS process %d", i); // Print the Process
        need[i][0] = -1;               // assign -1 if Process done
        y++;                           // cont if process done
      }

    } // end for loop

    if (y == m) { // if all done then
      p = 0;      // exit while loop
    }

  } // end while

  printf("\n");

}


/*

After Input of all required values 

OUTPUT RESULT

Alocation Values :
 	 0 	 0 	 1 	 2
 	 1 	 0 	 0 	 0
 	 1 	 3 	 5 	 4
 	 0 	 6 	 3 	 2
 	 0 	 0 	 1 	 4


Max Values :
 	 0 	 0 	 1 	 2
 	 1 	 7 	 5 	 0
 	 2 	 3 	 5 	 6
 	 0 	 6 	 5 	 2
 	 0 	 6 	 5 	 6


Need Values :
	 0	 0	 0	 0
	 0	 7	 5	 0
	 1	 0	 0	 2
	 0	 0	 2	 0
	 0	 6	 4	 2

 SS process 0
 SS process 2
 SS process 3
 SS process 4
 SS process 1


*/

```

Banker’s Algorithm is a resource allocation and deadlock avoidance Algorithms.

The Banker algorithm, sometimes referred to as the detection algorithm, is a resource allocation and deadlock avoidance algorithm developed by Edsger Dijkstra that tests for safety by simulating the allocation of predetermined maximum possible amounts of all resources, and then makes an "s-state" check to test for possible deadlock conditions for all other pending activities, before deciding whether allocation should be allowed to continue. 

Procedure: 
1. Input: 
We need input for no. of resources and procedures from user. Then, we need three matrices from user for allocation matrix, max matrix and available matrix.  

* Allocation Matrix: instances allocated to process 
* Max Matrix: Maximum instances required by process 
* Available Matrix: Matrix available for resources 

2. Process: 
First, we will create a need matrix, this matrix will show us instances of resources needed more by process.  
- Need Matrix = Max Matrix – Allocation Matrix 

Then, for every process we need to compare every instance of need matrix with every consecutive instance of available matrix. If every pair of need matrix instance is less or equal to instance of available matrix, then only we will allocate resource to that process. And hence, the process will be done.  

Need Matrix <= Available Matrix 

At last if all instances of need matrix are less than equal to available matrix, then only process will run. Then as the process ends, new available resources will be assigned.  

- New Available Matrix = Allocation Matrix + Available Matrix 


3. Output : Safe Sequence of Process 

## Another C impl
```c

#include <stdio.h>
#include<stdlib.h>
#include <pthread.h>
#define NUMBER_OF_CUSTOMERS 5
#define NUMBER_OF_RESOURCES 3

int total[NUMBER_OF_RESOURCES]={0};
int available[NUMBER_OF_RESOURCES];
int maximum[NUMBER_OF_CUSTOMERS][NUMBER_OF_RESOURCES];
int allocation[NUMBER_OF_CUSTOMERS][NUMBER_OF_RESOURCES]={0};
int need[NUMBER_OF_CUSTOMERS][NUMBER_OF_RESOURCES];
int Bavailable[NUMBER_OF_RESOURCES];
int Bmaximum[NUMBER_OF_CUSTOMERS][NUMBER_OF_RESOURCES];
int Ballocation[NUMBER_OF_CUSTOMERS][NUMBER_OF_RESOURCES];
int Bneed[NUMBER_OF_CUSTOMERS][NUMBER_OF_RESOURCES];
int Finish[NUMBER_OF_CUSTOMERS]={0};

int release_resources(int customer_num);
int request_resources(int customer_num, int request[]);
void *thread_func(void* customer_numt);
int bankerAlgorithm(int customer_num,int request[]);
void printState();
pthread_mutex_t mutex;
char string[NUMBER_OF_RESOURCES*2]={0};
int safeSequence[NUMBER_OF_CUSTOMERS]={0};


int main(int argc, const char * argv[]) {
    
  //********************* initialize the matrices ****************
    for(int i=0 ;i<argc-1;i++){
        available[i]=atoi(argv[i+1]);
        total[i]=available[i];      // in the begining available resources equal to total resources
    }
    for(int i=0;i< NUMBER_OF_RESOURCES;i++){
        for(int j=0;j<NUMBER_OF_CUSTOMERS;j++){
            maximum[j][i]=rand()%(total[i]+1);     //maximum should less than total
            need[j][i]=maximum[j][i];    // need=maximum-allocation  (allocation=0)
        
        }
    
    }
    //********************* initialize the matrices ****************
    
    //****************** print the state of the process *********************
    
    for (int i=0; i<NUMBER_OF_RESOURCES; i++) {
        string[i*2]=i+'A';
        string[i*2+1]=' ';
    }
    printf("Total system resources are:\n");
    printf("%s\n",string);
    for (int i=0; i<NUMBER_OF_RESOURCES; i++) {
        printf("%d ",total[i]);
    }
    printf("\n\nProcesses (maximum resources):\n");
    printf("   %s\n",string);
    for(int i=0; i<NUMBER_OF_CUSTOMERS;i++){
        printf("P%d ",i+1);
        for(int j=0;j<NUMBER_OF_RESOURCES;j++){
            printf("%d ",maximum[i][j]);
        }
        printf("\n");
    }

    printState();
   //****************** print the state of the process *********************
    
    
   pthread_mutex_init(&mutex, NULL);    //initialize mutex
   pthread_t p1,p2,p3,p4,p5;
   int a=0,b=1,c=2,d=3,e=4;
   pthread_create(&p1,NULL,thread_func,&a);
   pthread_create(&p2,NULL,thread_func,&b);
   pthread_create(&p3,NULL,thread_func,&c);
   pthread_create(&p4,NULL,thread_func,&d);
   pthread_create(&p5,NULL,thread_func,&e);     //create 5 threads
   char *returnV;
    
   pthread_join(p1,(void**)&returnV);
   pthread_join(p2,(void**)&returnV);
   pthread_join(p3,(void**)&returnV);
   pthread_join(p4,(void**)&returnV);
   pthread_join(p5,(void**)&returnV); // wait for all the 5 threads to terminate

   
    
    
    return 0;
}


void *thread_func(void* Tcustomer_num){
    
    
    int *c=(int*)Tcustomer_num;
    int customer_num= *c;
    
    int requestSum=0;
    
    while(!Finish[customer_num]){   // the whilie loop stops when the thread has finished and its need becomes zero
        requestSum=0;
        int request[NUMBER_OF_RESOURCES]={0};
        
        for(int i=0;i<NUMBER_OF_RESOURCES;i++){
           
            request[i]=rand()%(need[customer_num][i]+1); // generate a request below its need randomly
            requestSum=requestSum+request[i];
        }
        
        if(requestSum!=0)  // to make sure it doesn't requst for 0 reaources
            while(request_resources(customer_num,request)==-1); // only when the request has been granted succesfully
                                                                // will the loop terminates, otherwise it will keep
                                                                // making the same request.
        
        
    }
    
    
   
    
    return 0;
}



int request_resources(int customer_num, int request[]){
    
    
    
    int returnValue=-1;
    
    // since i want the process to print the result continuously and also run without race condition, i make the part
    // involing printing and accesing globale variable critical.
    //****************** critical section*********************
    pthread_mutex_lock(&mutex);
    
    printf("\nP%d requests for ",customer_num+1);
    for(int i=0;i<NUMBER_OF_RESOURCES;i++){
        printf("%d ",request[i]);
    }
    printf("\n");


        for(int i=0;i<NUMBER_OF_RESOURCES;i++){   // to check whether request<= availabe
                                                  // if it is not, then it will return -1
            if(request[i]>available[i]){
                printf("P%d is waiting for the reaources...\n",customer_num+1);

                pthread_mutex_unlock(&mutex);   // before the thread return, it has to unlock the mutex
                return -1;
            }
            
        }
        
   

    returnValue=bankerAlgorithm(customer_num,request);  // execute banker's algorithm
    
    if(returnValue==0){ 
        int needIsZero=1;
        printf("a safe sequence is found: ");
        for(int i=0;i<NUMBER_OF_CUSTOMERS;i++){
            printf("P%d ",safeSequence[i]+1 );
        
        }
        printf("\nP%d's request has been granted\n",customer_num+1);
       
        for(int j=0;j<NUMBER_OF_RESOURCES;j++){ // give the resources to the theread
            allocation[customer_num][j]=allocation[customer_num][j]+request[j];
            available[j]=available[j]-request[j];
            need[customer_num][j]=need[customer_num][j]-request[j];
            if(need[customer_num][j]!=0){// to check if need is zero
                needIsZero=0;
            }
        
        }
        
        if(needIsZero){
            
            Finish[customer_num]=1; // if need is zero, mark the thread "finish"
            release_resources(customer_num); // release resources when a thread finishes
            
        }
        
        printState();
        
    }
    else{
        printf("cannot find a safe sequence\n");
    
    }
   
    pthread_mutex_unlock(&mutex); // unlock the mutex
    return returnValue;
}


int release_resources(int customer_num){
    
    
    printf("P%d releases all the resources\n",customer_num+1);
    for(int j=0;j<NUMBER_OF_RESOURCES;j++){
        available[j]=available[j]+allocation[customer_num][j]; // release the resources
        allocation[customer_num][j]=0;
        
        
    }
    
    
    
    return 0;
}
int bankerAlgorithm(int customer_num,int request[]){
    int finish[NUMBER_OF_CUSTOMERS]={0};
    
    for(int i=0;i<NUMBER_OF_RESOURCES;i++){ // copy the matrices
        Bavailable[i]=available[i];
        for(int j=0;j<NUMBER_OF_CUSTOMERS;j++){
            Ballocation[j][i]=allocation[j][i];
            
            Bmaximum[j][i]=maximum[j][i];
         
            Bneed[j][i]=need[j][i];
        
        
        }
    }
    
        for(int i=0;i<NUMBER_OF_RESOURCES;i++){ // pretend to give the resource to the thread
            Bavailable[i]=Bavailable[i]-request[i];
            Ballocation[customer_num][i]=Ballocation[customer_num][i]+request[i];
            Bneed[customer_num][i]=Bneed[customer_num][i]-request[i];
        }
    
   

    
//*************************safety Algorithm***************************
    int count=0;
    while (1){
        
        int I=-1;
        
        for (int i=0; i<NUMBER_OF_CUSTOMERS; i++){ // to find a thread that its need is less than or equal to available.
            int nLessThanA=1;
            for (int j=0; j<NUMBER_OF_RESOURCES; j++){
                if (Bneed[i][j]>Bavailable[j] || finish[i]==1){
                    nLessThanA=0;
                    break;
                }
                
            }
            if (nLessThanA){ // if the thread is found, record its thread number
                I=i;
                break;
            }
            
        }
        
        if (I!=-1){
            safeSequence[count]=I; // record the sequence
            count++;
            finish[I]=1; // mark the thread "finish"
            for (int k=0; k<NUMBER_OF_RESOURCES; k++){  // pretend to give the reaource to thread
                Bavailable[k]=Bavailable[k]+Ballocation[I][k];
            }
            
            
        }
        else{ // if can not find any thread that its need is less than or equal to available.
            
            for (int i=0; i<NUMBER_OF_CUSTOMERS; i++){
                if (finish[i]==0){ // if there is any thread hasn't been found, that means it can't find a safe sequence
                    return -1;
                }
                
            }
            return 0;  // all the threads have been found
            
            
        }

    
    
    }
    
//*************************safety Algorithm***************************
    
    
}


void printState (){
    
    
    printf ("Processes (currently allocated resources):\n");
    printf ("   %s\n", string);
    
    for (int i=0; i<NUMBER_OF_CUSTOMERS; i++){
        printf ("P%d ", i+1);
        for (int j=0; j<NUMBER_OF_RESOURCES; j++){
            printf ("%d ", allocation[i][j]);
        }
        printf ("\n");
    }
    printf ("Processes (possibly needed resources):\n");
    printf ("   %s\n", string);
    for (int i=0; i<NUMBER_OF_CUSTOMERS; i++){
        printf ("P%d ", i+1);
        for (int j=0; j<NUMBER_OF_RESOURCES; j++){
            printf ("%d ", need[i][j]);
        }
        
        printf ("\n");
    }
    printf ("Available system resources are:\n");
    printf ("%s\n", string);
    
    for (int i=0; i<NUMBER_OF_RESOURCES; i++) {
        printf ("%d ", available[i]);
    }
    
    printf ("\n");
    
    
    
}


```

## Another Cpp impl
```cpp
#include <iostream>

#include <vector>
using namespace std;

void printResources(vector<int>available, vector<vector<int>>needed, vector<vector<int>>allocation)
{
 cout<<"Available resources:\n";
 char start='A';
 for(int i=0;i<available.size();i++){
    cout<<start<<" ";
    start++;
 }
 cout<<endl;
 for(int i=0;i<available.size();i++) cout<<available[i]<<" ";
 cout<<endl<<endl<<endl;

 cout<<"Needed resources:\n";
 start='A';
 for(int i=0;i<available.size();i++){
    cout<<start<<" ";
    start++;
 }
 cout<<endl;
 for(int i=0;i<needed.size();i++){
    for(int j=0;j<available.size();j++){
        cout<<needed[i][j]<<" ";
    }
    cout<<endl;
 }
 cout<<endl<<endl<<endl;
 cout<<"allocated resources:\n";
 start='A';
 for(int i=0;i<available.size();i++){
    cout<<start<<" ";
    start++;
 }
 cout<<endl;
 for(int i=0;i<allocation.size();i++){
    for(int j=0;j<available.size();j++){
        cout<<allocation[i][j]<<" ";
    }
    cout<<endl;
 }
  cout<<endl<<endl<<endl;
}

bool bankerAlogrithm(vector<int>available, vector<vector<int>>needed, vector<vector<int>>allocation,bool print) {
    pair<vector<int>, bool> result;

    int n = 5, m = 3, counter = 0;
    bool hasProcess = true;
    vector<int> done;
    bool isdone = false;

    while (true)//start of the alogrithm
    {
        int detect = 0;
        for (int i = 0; i < n; i++)
        { // processes

            for (vector<int>::iterator it = done.begin();
                it != done.end();
                it++)
            {
                if (*it == i)
                {
                    isdone = true;

                }
            }
            if (isdone == true) {
                isdone = false;
                continue;
            }


            for (int j = 0; j < m; j++)
            {                                    // resourses
                if (needed[i][j] > available[j]) // avilable is not enough
                {
                    hasProcess = false;
                    break;
                }
            }

            if (hasProcess == true) // got all it wants
            {
                detect++;//counter to know if any process got what it want in this loop
                counter++;//counter for how many proccesses has ended
                done.push_back(i);
                for (int j = 0; j < m; j++)
                {
                    available[j] += allocation[i][j];
                    allocation[i][j]=0;
                    needed[i][j]=0;
                }
                if(print==true){
                    cout<<endl<<"p"<<i<<" has done."<<endl;
                    printResources(available,needed,allocation);
                }
                i = -1;
            }
            hasProcess = true;


        }
        if (counter >= n)
        {
            if(print==true)cout<<"no Deadlock."<<endl;
            return true;
        }
        if (detect == 0) {
            if(print==true)cout<<"There is Deadlock!!."<<endl;
            return false;
        }
    }

}

void recover(vector<int>&available, vector<vector<int>>&needed, vector<vector<int>>&allocation) {
    int processNo = 0;
    int max = 0;
    int sum = 0;
    while (!bankerAlogrithm(available, needed, allocation,false))
    {
        for (int i = 0; i < needed.size(); i++)
        {
            for (int j = 0; j < available.size(); j++)
            {
                sum += allocation[i][j];
            }
            if (sum > max)
            {
                max = sum;
                processNo = i;
            }
            sum = 0;
        }
        for (int j = 0; j < available.size(); j++)
        {
            available[j] += allocation[processNo][j];
            needed[processNo][j] += allocation[processNo][j];
            allocation[processNo][j] = 0;
        }
        cout<<endl<<"P"<<processNo<<" has Released\n"<<endl;
        printResources(available,needed,allocation);
        bankerAlogrithm(available, needed, allocation,true);
    }

}

int main()
{
    int n = 5, m = 3;
    vector<int> available(m);
    vector<vector<int>> needed(n, vector<int>(m, 0));
    vector<vector<int>> maximum(n, vector<int>(m, 0));
    vector<vector<int>> allocation(n, vector<int>(m, 0));

    bool safe;


    cout << "Enter the initial number of available resources (space seperated):\n";
    for (int i = 0; i < m; i++)
    {
        cin >> available[i];
    }
    cout << "Enter the maximum need for each process in a seperate line:\n";
    for (int i = 0; i < n; i++)
    {

        for (int j = 0; j < m; j++)
        {
            cin >> maximum[i][j];
        }
    }
    cout << "Enter the allocated resources for each process in a seperate line:\n";
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            cin >> allocation[i][j];
            available[j] -= allocation[i][j];
        }
    }

    //calculate needed
    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            needed[i][j] = maximum[i][j] - allocation[i][j];
        }
    }

    safe = bankerAlogrithm(available, needed, allocation,true);

    //taking commands
    string command;
    int process, * resources = new int[m];
    while (true)
    {
        cout << "Enter command:\n";
        cin >> command;
        if (command == "Quit")
        {
            return 0; // terminate program
        }
        else if (command == "Recover")
        {
            if (!safe)
            {
                recover(available, needed, allocation);
                safe=true;
            }
            else
            {
                cout << "System already safe, no need to recover\n";
            }
            continue;
        }
        if(command!="RQ"||command!="RL"){cout<<"Invalid Command.!"<<endl;continue;}

        cin >> process;
        for (int i = 0; i < m; i++)
        {
            cin >> resources[i];
        }
        //check safe state
        if (!safe)
        {
            cout << "System has deadlock, please recover first\n";
            continue;
        }

        if (command == "RQ")
        {
            bool isValid = true;
            string message;
            // check request <= needed
            for (int i = 0; i < m; i++)
            {
                if (resources[i] > needed[process][i])
                {
                    isValid = false;
                    message = "ERROR: process has exceeded its maximum claim\n";
                }
            }
            //check request <= available
            for (int i = 0; i < m; i++)
            {
                if (resources[i] > available[i])
                {
                    isValid = false;
                    message = "ERROR: not enough resources available\n";
                }
            }
            if (!isValid)
            {
                cout << message;
                continue;
            }
            //request can be granted
            for (int i = 0; i < m; i++)
            {
                allocation[process][i] += resources[i];
                needed[process][i] -= resources[i];
                available[i] -= resources[i];
            }
            //execute safety algorithm
            safe = bankerAlogrithm(available, needed, allocation,true);
        }
        else if (command == "RL")
        {
            bool isValid = true;
            //check release <= allocation
            for (int i = 0; i < m; i++)
            {
                if (resources[i] > allocation[process][i])
                {
                    cout << "ERROR: resources exceeded allocation\n";
                    isValid = false;
                    break;
                }
            }
            if (! isValid)
            {
                continue;
            }
            // release granted
            for (int i = 0; i < m; i++)
            {
                available[i] += resources[i];
                needed[process][i] += resources[i];
                allocation[process][i] -= resources[i];
            }
            safe = bankerAlogrithm (available, needed, allocation, true);
        }
    }
}
```

(An algorithm for deadlock avoidance in operating systems)

Advanced Operating Systems (CS342) - Assignment (01) - Chapter (07) - Deadlock

It is needed to be familiar with the Deadlock problem and how to solve it. Therefore, it is required to apply the deadlock avoidance algorithms by applying the Banker's algorithm. You will write a Java code that implements the banker’s algorithm: processes request and release resources and the banker will grant a request only if it leaves the system in a safe state. A request is denied if it leaves the system in an unsafe state. The bank will employ the strategy outlined in the textbook & lectures whereby it will consider requests from n processes for m resources. The bank will keep track of the resources using the following data structures:

int [] available; //the available amount of each resource

int [][] maximum; //the maximum demand of each process

int [][] allocation; //the amount currently allocated to each process

int [][] need; //the remaining needs of each process

You should build a test program to test your code. The test program should take the initial number of the available resources at the bank, the maximum need, and the actually allocated resources for each process. Your program should calculate the need matrix as well as the new available vector.

Output:

The output screen must appear every action happened as well as the current state for the bank by showing the values for your data structures as well as the process requests and releasing forthe resources. The bank decisionsto either deny or approve the requests must be shown through the output screen.

After the testing the user can type:

RQ <process#> : It means that process request resources. So add this request to the needed resources for the given process and check again if the system is in a safe state.

RL <process#> : It means that process release resources. So check if release resources are less than or equal to allocated resources and if so, then subtract this release resources from allocated resources for a given process.

Recover: if a request let to a Deadlock, apply recovery algorithm (choose a victim , force it to release resources and check again if the system is in a safe state, if still in an unsafe state then will choose a victim again force it to release and check again and keep repeating until it reaches a safe state).

Mention based on what did you choose your victim (i.e. priority, how much longer to complete, resources used, resources needed …etc.).

Quit: it closes the application