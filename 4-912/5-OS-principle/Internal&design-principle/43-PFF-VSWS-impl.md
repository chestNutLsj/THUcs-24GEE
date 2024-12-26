
## impl in C
```c
// PFF & VSWS
// Input: file of memory access simulation
// Author: Weichen Xu
// Email: wx431@nyu.edu
// Date: 12/09/2015

/*
 *  I use a test case which considers locality & transient, did not upload the file generating the test case
 *
 *  In PFF
 *	Generally, page fault number decreases when F(reference time) increases
 *  Initially, when F is 1, the page fault number is at maximum, MAX = all mem reference number - continuous access at same address
 *							because page fault can only be avoided when same memory page be accessed in adajcent order
 *             when F is small, the degrowth of page fault is rapid
 *             when F gets bigger, the degrowth of page fault slows down
 *             when F reaches some point, the page fault number is at minimum, MIN = the number of the pages this program occupies
 *	            
 *             when page fault number decreases, the number of frames allocated increases
 *
 *  In VSWS
 *	Generally, page fault number decreases when M increase
 *			   page fault number decreases when L increase
 *             page fault number decreases when Q increase
 *
 *             when page fault number decreases, the number of frames allocated increases
 *
 *  Considering the page fault times in total, PFF is better than VSWS
 *  Considering the frames allocated,  min_frame_allocated_frequency & max_frame_allocated, VSWS is better
 *  	       in other words, VSWS keeps much fewer frames in memory, especially between program transients.
 *  Overall, I think it is a tradeoff because PFF avoid more page faults, which is faster, while VSWS allows more runnable programes
 *             which might avoid CPU wasting
 */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define F 30
#define M 90
#define Q 20
#define L 150

#define MAX_FILE_NAME_LENGTH 10
#define MAX_MEM_LENGTH	10
#define MAX_MEM_ACCESS_TIME	50000
#define MIN_FRAME_NUMBER 30

typedef struct{
	int address;
	int use_bit;
	bool in_memory;	// true: in residnet set
}Page;

int memAccess(Page *mem_pages, int *mem_acc, int mem_acc_times, int replace_mode);
int loadMemAccessFile(int **mem, int *mem_acc_times);
void shrinkResidentSet(Page *mem_pages, int mem_pages_size);
int PFF(Page *mem_pages, int *mem_pages_size, int page_addr, int cur_reference_time, int *last_reference_time);
int VSWS(Page *mem_pages, int *mem_pages_size, int page_addr, int cur_reference_time, int *last_reference_time, int *elapsed_page_faults);

//int pageReplaceMent(int *mem_acc, int mem_acc_times, int replace_mode);

// get the file name from input
// load the mem access order into array *mem
// Input: **mem: the order of mem access
//        *mem_acc_times: the number of mem access
// Output: how many pages the process occupies, the 1st line of the file
int loadMemAccessFile(int **mem_acc, int *mem_acc_times){
	char fileName[MAX_FILE_NAME_LENGTH];
	char buffer[MAX_MEM_LENGTH];
	char *mode ="r";
	FILE *fp;
	// mem_access_times: the times of memory access by the process
	// pages_num: the number of the pages that process occupies
	int mem_access_times = 0, pages_num;	
	int mem_access[MAX_MEM_ACCESS_TIME];
	
	printf("Enter the file name:");
	scanf("%s", fileName);
	fp = fopen(fileName, mode);
	if(fp == NULL){
		printf("Can not open %s\n", fileName);
		exit(1);
	}
	else printf("Loading mem_acc file\n");

	// get ints line by line
	// load first line to be pages_num
	if(fgets(buffer, MAX_MEM_LENGTH, fp) != NULL){
		sscanf(buffer, "%d\n", &pages_num);
	}
	// the remains to be the mem access sequences
	while(fgets(buffer, MAX_MEM_LENGTH, fp) != NULL){
		sscanf(buffer, "%d\n", &mem_access[mem_access_times]);
		//printf("%d\n", mem_access[mem_access_times]);
		mem_access_times ++;
	}
	// allocate space for *mme and copy data to it
	*mem_acc_times = mem_access_times;
	*mem_acc = (int *)malloc(sizeof(int) * (mem_access_times));
	for(int i=0; i<mem_access_times; i++){
		(*mem_acc)[i] = mem_access[i];
	}
	fclose(fp);
	return pages_num;
}

// shrink resident set size by discarding pages whose use-bit == 0
// set use-bit = 0 with use-bit = 1 before
// *mem_pages: all mem pages of the process, use in_memory to mark resident set instead of dynamic allocation
// mem_pages_size: size of mem_pages
void shrinkResidentSet(Page *mem_pages, int mem_pages_size){
	for(int i=0; i<mem_pages_size; i++){
		// if use-bit is 0, discard
		if(mem_pages[i].use_bit == 0){ 
			mem_pages[i].in_memory = false;
		}
		// otherwise, set from 1 to 0
		else{
			mem_pages[i].use_bit = 0;
		}	
	}
}

// count how many pages are in memory, in other words, the size of resident set
int getResidentSetSize(Page *mem_pages, int mem_pages_size){
	int count = 0;
	for(int i=0; i<mem_pages_size; i++){
		if(mem_pages[i].in_memory)	count++;
	}
	return count;
}
//  simulate the memory access by PFF resident algo
//  restore the page_fault_number in caller
//  return 0: no page fault
//  return 1: page fault

int PFF(Page *mem_pages, int *mem_pages_size, int page_addr, int cur_reference_time, int *most_recent_page_fault_time){
	// the page index in memory set
	int memory_set_index = -1;
	// page fault time
	int pff; 
	// whether page in resident set
	for(int i=0; i<*mem_pages_size; i++){
		if(mem_pages[i].address == page_addr){ 
			memory_set_index = i;
			if(mem_pages[i].in_memory == true) {
				mem_pages[i].use_bit = 1; // set use-bit to 1 when access
				return 0;
			}
			else break;
		}
	}
	// page fault happens
	// if the page is not in the memory set, add it
	if(memory_set_index<0){
		mem_pages[*mem_pages_size].address = page_addr;
		mem_pages[*mem_pages_size].in_memory = true;	// put in resident set
		mem_pages[*mem_pages_size].use_bit = 1;	// set use-bit to 1
		memory_set_index = *mem_pages_size;	
		(*mem_pages_size)++;
	}
	// get the time between two page faults
	pff = cur_reference_time-*most_recent_page_fault_time;
	// set the most recent reference time to current
	*most_recent_page_fault_time = cur_reference_time;
	//printf("pff:%d\n",pff);
	if(pff < F){
		// add to the resident
		// set the use-bit to 1
		mem_pages[memory_set_index].in_memory = true;
		mem_pages[memory_set_index].use_bit = 1;
	}
	else{
		// shrink resident set size by cleaning pages whose use-bit == 0
		// set use-bit = 0 to those pages with use-bit = 1 before
		shrinkResidentSet (mem_pages, *mem_pages_size);
	}
	return 1;
}

// simulate memory access by VSWS
// *mem_page: current memory set, *mem_pages_size: current memory set size
// page_addr: address for memory
// return 1 if page fault occurs
// return 0 otherwise

int VSWS (Page *mem_pages, int *mem_pages_size, int page_addr, int cur_reference_time, int *most_recent_sampling_time, int *elapsed_page_faults){
	// find the index of the page in memory set
	int memory_set_index = -1, page_fault_occur = 1;
	int elapsed_time = cur_reference_time - (*most_recent_sampling_time);
	// whether page in resident set
	for (int i=0; i<*mem_pages_size; i++){
		if (mem_pages[i]. address == page_addr){ 
			memory_set_index = i;
			if (mem_pages[i]. in_memory == true) {
				page_fault_occur = 0;
				mem_pages[i]. use_bit = 1; // set use-bit to 1 when access
			}
			break;
		}
	}

	// if page_fault_occur
	// add elapsed_page_faults by one
	if (page_fault_occur != 0)	(*elapsed_page_faults)++;
	// add this page to the memory set if it not found
	if (memory_set_index<0){
		mem_pages[*mem_pages_size]. address = page_addr;
		mem_pages[*mem_pages_size]. in_memory = true;	// put in resident set
		mem_pages[*mem_pages_size]. use_bit = 1;	// set use-bit to 1
		memory_set_index = *mem_pages_size;	
		(*mem_pages_size)++;
	}

	// if elapsed time reaches L, suspend and scan
	// discard all pages whose use-bit = 0
	// set all remain pages use-bit = 1
	if (elapsed_time >= L){
		shrinkResidentSet (mem_pages, *mem_pages_size);
		*most_recent_sampling_time = cur_reference_time;
		*elapsed_page_faults = 0;
	}
	// if eplapsed_page_faults >= Q and elapsed_reference_time >= M
	// suspend and scan
	else if ((*elapsed_page_faults) >= Q && elapsed_time >= M){
		shrinkResidentSet (mem_pages, *mem_pages_size);
		*most_recent_sampling_time = cur_reference_time;
		*elapsed_page_faults = 0;
	}
	return page_fault_occur;
}

// Simulate the process of page fetch & resident set
// *mem_pages: represent the current memory set of the program
//             
// *mem_acc: the sequence of mem access
// mem_acc_times: the number of mem accesss
// replace_mode: 1: for PFF replacement algo
//               2: for VSWS replacement algo
// return the (total) number of page faults
int memAccess (Page *mem_pages, int *mem_acc, int mem_acc_times, int replace_mode){
	int mem_pages_size = 0;
	int page_fault_times = 0;
	// for PFF
	int most_recent_page_fault_time = 0; 
	// for VSWS
	int elapsed_page_faults = 0, most_recent_sampling_time = 0;
	// resident set size statistic, allocated frames
	int max_frame_allocated = 0, min_frame_count = 0, current_frame_allocated;
	float min_frame_frequency = 0.0;
	for (int i=0; i<mem_acc_times; i++){
		// require the i th page according to replacement algo
		if (replace_mode == 1)	{
			// if page fault happpend, page_fault_times ++,  set most_recent_page_fault_time to current
			if (PFF (mem_pages, &mem_pages_size, mem_acc[i], i+1, &most_recent_page_fault_time)){
				page_fault_times++;
			}
		}
		if (replace_mode == 2)	{
			// if page fault happpend, page_fault_times ++
			// if scan the resident set, set most_recent_sampling_time to current
			if (VSWS (mem_pages, &mem_pages_size, mem_acc[i], i+1, &most_recent_sampling_time, &elapsed_page_faults)){
				page_fault_times++;
			}
		}
		// get the number of frames allocated
		current_frame_allocated = getResidentSetSize (mem_pages, mem_pages_size);
		if (current_frame_allocated < MIN_FRAME_NUMBER)	min_frame_count ++;
		max_frame_allocated = max_frame_allocated>current_frame_allocated? max_frame_allocated: current_frame_allocated;
	}
	min_frame_frequency = (float) min_frame_count/(float) mem_acc_times;
	printf ("Min frame frequency: %f\n", min_frame_frequency);
	printf ("Max frame allocated: %d\n", max_frame_allocated);
	printf ("Page fault total: %d\n", page_fault_times);
	return page_fault_times;
}

int main (){
	int *mem_acc = NULL, pages_num, mem_acc_times, page_fault_times = 0;
	Page *mem_pages = NULL;
	pages_num = loadMemAccessFile (&mem_acc, &mem_acc_times);
	// used to test the threshold
	//for (int i=1; i<10; i++){
	//	M = 90;
	//	Q = 40;
	//	L = 150;
		printf ("Using PFF\n");
		mem_pages = (Page *) malloc (sizeof (Page) * pages_num);
		page_fault_times = memAccess (mem_pages, mem_acc, mem_acc_times, 1);
		if (mem_pages) free (mem_pages);
		printf ("Using VSWS\n");
		mem_pages = (Page *) malloc (sizeof (Page) * pages_num);
		page_fault_times = memAccess (mem_pages, mem_acc, mem_acc_times, 2);
		if (mem_pages) free (mem_pages);
	//}
	//printf ("Memory access times: %d\n", mem_acc_times);
	
	if (mem_acc)	free (mem_acc);
	
}
```

## impl in C++
```cpp
/*rrk310*/
//Assignment 3 - "Performance analysis of PFF and VSWS page replacement algorithms"
#include<iostream>
#include<vector>
#include<fstream>
#include<map>
#include<climits>

using namespace std;

class PFF //PageFault Frequency
{
	class Result //Holds Results for very 'F' run
	{
		public:
			int faults; //Track num page faults occuring after every memory reference
			int numOfNewFrames; //Track num of new frames alloc'ed
			int maxResidentSize;//Tracks how big the number of resident pages were at one point during simulation
			int lessThanTenFrames;//Tracks num of times , used frames are less than 10
			Result()
			{
				faults =0;
				numOfNewFrames =0;
				maxResidentSize=0;
				lessThanTenFrames=0;
			}
			void disp()
			{
				cout<<"max page faults "<<faults<<" max new frames alloc'ed "<<numOfNewFrames<<" maxResidentSize "<<maxResidentSize<<" lessThanTenFrames count "<<lessThanTenFrames<<endl;
			}
	};
	map<int, Result > recordedResults; //Record per F value
	void pffOneRun(Result &result ,vector<int> &references ,int fval ,int resSize)
	{
		int run=0;//virtual time
		int lastPageFault=run;
		int numOfPageFaults =0;
		int numOfNewFrames =0;
		int currResidentSize=0;
		int maxResidentSize =INT_MIN;
		int maxPages = resSize;//max pages that can be occupied by the process with use bit.
		map<int,bool> currentPages;
		//This map is dynamic , so if key contains - it means we have a frame available(it can be occupied or not) , 
		//if not then a new frame is like adding a <key,val> pair
		for(vector<int>::iterator iter = references.begin(); iter!=references.end();iter++)
		{
			run++;
			int markedPages = retCurrentMarkedPages(currentPages);
			if(markedPages <10)
				result.lessThanTenFrames++;
			if(markedPages == maxPages) //sanity check
			{
				cout<<"our fixed size of max pages for this process has exceeded so cant accomodate any more "<<markedPages<<endl;
				break;
			}	
			if(currentPages.find(*iter) != currentPages.end())
			{
				currentPages[*iter] = true;//mark active it means there is a frame as well
			}
			else
			{
				numOfPageFaults++;
				if(run - lastPageFault > fval)
				{
					//Now remove the free pages
					vector<int> toBeDeleted;
					for (map<int,bool>::iterator iter = currentPages.begin() ;iter!=currentPages.end(); iter++ )
					{
						if((*iter).second == false)//fetch pages not accessed
							toBeDeleted.push_back((*iter).first);
						else
							currentPages[(*iter).first] = false;//mark them inactive if there are active
					}
					for (vector<int>::iterator iter = toBeDeleted.begin() ;iter!=toBeDeleted.end() ; iter++)
					{
						currentPages.erase(*iter);  
						currResidentSize--;
					}
				}
				else
				{
					allocateNewFrame();
					numOfNewFrames++;
					currentPages[*iter] = true;//Since we have are using a dynamic array , no need to allocate new frame .Just make sure of sanity.
					currResidentSize++;
					if(maxResidentSize < currResidentSize)
						maxResidentSize = currResidentSize;
				}
				lastPageFault = run;//record this page fault
			}
		}
		result.faults = numOfPageFaults;
		result.numOfNewFrames = numOfNewFrames;
		result.maxResidentSize = maxResidentSize;
	}

	void allocateNewFrame(){} //dummy function to signify new frame allocation

	int retCurrentMarkedPages(map<int,bool> &currentPages)//returns pages in use.
	{
		int count =0;
		for(map<int,bool>::iterator iter = currentPages.begin(); iter!=currentPages.end();iter++)
		{
			if((*iter).second == true)
			{
				count++;
			}
		}
		return count;
	}
public:
	void startPFF (int resSize ,vector<int> &references)
	{
		for (int Fvalue = 1 ; Fvalue <=20 ; Fvalue++)//taking arbitrary numbers from 1 -20 just to prove how fvalue can reduce page faults.
		{
			Result result;
			pffOneRun (result, references, Fvalue, resSize);
			recordedResults.insert (make_pair<int, Result>(Fvalue, result));
		}
	}

	void displayResults ()
	{
		int minPageFaults =INT_MAX;
		int minResidentSize =INT_MAX;
		int Fvalue = -1;
		int maxNewFrames = INT_MIN;
		for (map<int,Result>:: iterator iter = recordedResults.begin () ;iter!=recordedResults.end (); iter++ )
		{
			cout<<"for Fvalue "<<(*iter). first<<endl;
			(*iter). second.disp ();
			if ((*iter). second. numOfNewFrames > maxNewFrames )
				maxNewFrames = (*iter). second. numOfNewFrames;
			if ((*iter). second. faults < minPageFaults)
			{
				minPageFaults = (*iter). second. faults;
				Fvalue = (*iter). first;
				minResidentSize = (*iter). second. maxResidentSize;
			}
		}
		cout<<"optimal F value accross runs is Fvalue "<<Fvalue<<" page faults "<<minPageFaults<<" min resident size "<<minResidentSize<<endl;
		cout<<"the max new frames alloced amongst all runs is "<<maxNewFrames<<endl;
	}
};

class VSWS
{
public:
	class Sample
	{
	public:
		int M; //min length of sampling interval
		int L; //max length of sampling interval
		int Q; //num faults in one sampling instance
		Sample (int M, int L, int Q)
		{
			this->M = M;
			this->L = L;
			this->Q = Q;
		}
		void display ()
		{
			cout<<"For Sample M "<<M<<" L "<<L<<" Q "<<Q<<endl;
		}
	};

	class SResult //Holds Results for very sampling run
	{
		public:
			int faults; //Track num page faults occuring after every memory reference
			int numOfNewFrames; //Track num of new frames alloc'ed
			int maxResidentSize;//Tracks how big the number of resident pages were at one point during simulation
			int lessThanTenFrames;//Tracks num of times , used frames are less than 10
			Sample &sample;//Holds a ref to sample data
			SResult (Sample &sample): sample (sample)
			{
				faults =0;
				numOfNewFrames =0;
				maxResidentSize=0;
				lessThanTenFrames=0;
			}
			void disp ()
			{
				sample.display ();
				cout<<"max page faults "<<faults<<" max new frames alloc'ed "<<numOfNewFrames<<" maxResidentSize "<<maxResidentSize<<" lessThanTenFrames count "<<lessThanTenFrames<<endl;
			}
	};

private:
	vector<SResult > recordedResults; //Record per F value
	vector<Sample > samples; //all samples

	void vswsOneRun (SResult &result ,vector<int> &references ,int resSize ,Sample &sample)
	{
		int M = sample. M;
		int L = sample. L;
		int Q = sample. Q;
		int currVirtualTime = 1;
		int numSamples =0 ;
		int maxPages = resSize;//max pages that can be occupied by the process with use bit.
		int numOfNewFrames =0;
		map<int,bool> currentPages; 
		//This map is dynamic , so if key contains - it means we have a frame available (it can be occupied or not) , 
		//if not then a new frame is like adding a <key,val> pair

		int minResidentSize = INT_MAX; //stores min residents for this run
		int maxResidentSize = INT_MIN;//stores max residents for this run
		int numFaults =0; //stores num of page faults encountered.
		int perSampleFault =0;
		int runTime = L;
		int evaluatedPerSampleWorkingSet=INT_MIN; //will store the max of all per samples , another approach can be to take mean.
		for (vector<int>:: iterator iter = references.begin (); iter!=references.end (); iter++)
		{
			if (currVirtualTime == 1)//begin interval
			{
				runTime = L;
				unMarkAllPages (currentPages);
			}
			if (currVirtualTime == runTime)//end interval - runTime can be bounded by M or L based on current Q
			{
				numSamples++;
				numFaults+=perSampleFault;
				currVirtualTime=1;
				perSampleFault=0;
				int currWorkingSet = retCurrentMarkedPages (currentPages);
				if (currWorkingSet >evaluatedPerSampleWorkingSet)
					evaluatedPerSampleWorkingSet = currWorkingSet;
				removeFreePages (currentPages);// release unreferenced pages
			}
			else
			{
				if (currentPages.find (*iter) != currentPages.end ())
				{
					currentPages[*iter] = true;//mark active - it means there is a frame as well.
				}
				else
				{
					numOfNewFrames++; //Here no need to check if there is a frame available since its dynamic
					currentPages[*iter] = true; //Since we have are using a dynamic array , no need to allocate new frame .Just make sure of sanity.
					//page fault !
					perSampleFault++;
				}
				currVirtualTime++;
				int markedPages = retCurrentMarkedPages (currentPages);
				if (markedPages <10)
					result. lessThanTenFrames++;
				if (markedPages == maxPages) //sanity check
				{
					cout<<"our fixed size of max pages for this process has exceeded so cant accomodate any more "<<markedPages<<endl;
					break;
				}
				if (perSampleFault >= Q)
				{
					if (currVirtualTime < M)
					{
						runTime = M;
					}
					else
					{
						currVirtualTime = runTime; //will stop the current sample to determine use bits
					}
				}
			}
		}
		result. faults = numFaults;
		result. numOfNewFrames = numOfNewFrames;
		result. maxResidentSize = evaluatedPerSampleWorkingSet > maxResidentSize ? evaluatedPerSampleWorkingSet: maxResidentSize;
	}

	void unMarkAllPages (map<int,bool> &currentPages) //un marks all occupied pages 
	{
		for (map<int,bool>:: iterator iter =currentPages.begin (); iter!=currentPages.end () ;iter++)
		{
			(*iter). second = false;
		}
	}

	void removeFreePages (map<int,bool> &currentPages) //removes pages that are free from resident set
	{
		vector<int> toBeDeleted;
		for (map<int,bool>:: iterator iter = currentPages.begin () ;iter!=currentPages.end (); iter++ )
		{
			if ((*iter). second == false)//fetch pages not accessed
				toBeDeleted. push_back ((*iter). first);
		}
		for (vector<int>:: iterator iter = toBeDeleted.begin () ;iter!=toBeDeleted.end () ; iter++)
		{
			currentPages.erase (*iter);  
		}
	}

	int retCurrentMarkedPages (map<int,bool> &currentPages) //returns pages in use.
	{
		int count =0;
		for (map<int,bool>:: iterator iter = currentPages.begin (); iter!=currentPages.end (); iter++)
		{
			if ((*iter). second == true)
			{
				count++;
			}
		}
		return count;
	}

	void allocateNewFrame (){} //dummy function to signify new frame allocation
public:
	VSWS ()
	{
		//run on these samples
		samples. push_back (Sample (1,4,1));
		samples. push_back (Sample (8,16,4));
		samples. push_back (Sample (8,20,6));
		samples. push_back (Sample (12,30,4));
	}

	void startVSWS (int resSize ,vector<int> &references)
	{
		for (vector<Sample>:: iterator iter = samples.begin (); iter!=samples.end () ;iter++)
		{
			SResult result (*iter);
			vswsOneRun (result, references, resSize, *iter);
			recordedResults. push_back (result);
		}
	}

	void displayResults ()
	{
		int minPageFaults =INT_MAX;
		int minResidentSize =INT_MAX;
		int maxNewFrames = INT_MIN;
		int index = -1;
		int found = -1;
		for (vector<SResult>:: iterator iter = recordedResults.begin () ;iter!=recordedResults.end (); iter++ )
		{
			index++;
			(*iter). disp ();
			if ((*iter). numOfNewFrames > maxNewFrames )
				maxNewFrames = (*iter). numOfNewFrames;
			if ((*iter). faults < minPageFaults)
			{
				minPageFaults = (*iter). faults;
				minResidentSize = (*iter). maxResidentSize;
				found = index;
			}
		}
		cout<<"optimal values are page faults "<<minPageFaults<<" min resident size "<<minResidentSize<<" for below sample "<<endl;
		if (found !=-1) recordedResults[found]. sample.display ();
		cout<<"the max new frames alloced amongst all runs is "<<maxNewFrames<<endl;
	}
};

int startSimulation (string &fileName)
{
	vector<int> pageReferences;
	int currResidentSize;
	int ret =0;
	ifstream memFile (fileName);
	if (! memFile)
	{
		ret = -1;
		cout<<"fopen failed!"<<endl;
	}
	else
	{
		string currSize;
		string references;
		int numReferences=0;
		getline (memFile, currSize);
		currResidentSize = atoi (currSize. c_str ());
		cout<<"resident size - "<<currResidentSize<<endl;
		while (getline (memFile, references))
		{
			pageReferences. push_back (atoi (references. c_str ()));
			numReferences++;
		}
		cout<<"num memory refs - "<<numReferences<<endl;
	
		cout<<"*********** begin PFF simulation ***********"<<endl;
		PFF pff;
		pff.startPFF (currResidentSize, pageReferences);
		pff.displayResults ();
		//After finding the results on a run of 129 page references , it was found that optimal value for f{1-20} was 12 
		//and the number page faults were 42 as opposed to 60 with f value =1
		//also there was a considerable improvement with resident size of 38 there by proving that the page fault rate decreases 
		//as the resident set increases.
		//In the dataset after introducing page references of different locality , it was found that the max resident size increased for a while.
		cout<<"*********** end PFF simulation **************"<<endl;


		cout<<"*********** begin VSWS simulation ***********"<<endl;
		VSWS vsws;
		vsws.startVSWS (currResidentSize, pageReferences);
		vsws.displayResults ();
		//After running the VSWS on the same dataset of 129 page references, it was found that a better minimum fault rate of 39 was acheived
		//for this sample M 12 L 30 Q 4 and a better resident size of 11.
		//Based on the max resident size , its clear that even with a less resident page , lesser fault rate can be acheived.
		//The algorithm was also run on page references of different locality and produces more or less same result.
		//On comparison it can be seen VSWS algorithm produces much lesser fault rate on an average provided a good sample is provided for that instant.
		cout<<"************ end VSWS simulation ***************"<<endl;
	}
	return ret;
}

int main (int argc ,char **argv)
{
	string fileName;
	int ret =0;
	cout<<"Enter file name : "<<endl;
	cin>>fileName;
	cout<<"File name - "<<fileName<<endl;
	ret = startSimulation (fileName);
	if (ret != 0)
		cout<<"simulation error!"<<endl;
	return 0;
}
```

## impl in Python
### createInputFile.py
```python
# createInputFile.py
#Name			: RUPANTA RWITEEJ DUTTA
#Date of Creation	: 12.12.2015

#This Program creates a input file named "Input.txt" to be used as an input in the Assignment3.py program.
import random

#Function to genrate input file to be read
fo = open ('Input.txt','w')				# Create and Open File "Input.txt" in Write Mode
fo.write('30')						# Write '30' in the First Line, Indicating the Number of Pages the Process Refers

for num in range (1,10001):				# Generate 10000 Numbers in the Range 1-30
	var = random.randint(1,30)			# Write Each Number on a Different Line
	var = '\n' + str(var)	
	fo.write(var)

fo.close()						# Close File
```

### Assignment.py
```python
# Name			: RUPANTA RWITEEJ DUTTA
# Date of Creation	: 12.12.2015

# This Program takes a file as input on the command line and calculates number of page faults by implementing the PFF 
#   and VSWS Algorithms.
import sys

# Function to read input file
def readFile(fileName):
	try:
		fo = open(fileName,'r')
	except:
		print "\nERROR: Unable to Open Input File: '%s'\nPlease Check: Path/FileName" %fileName
		print "----------------------------------------------\n"
		exit(0)
	else:	
		data = fo.read()
		attributes = data.split("\n")
		return attributes
		fo.close()

# Function to Implement PFF
#-------------------------------------------------------------------
# Sample Outputs:
# 	Number of Pages the Process Occupies	: 30
#	Total Number of Page References		: 10000
#-------------------------------------------------------------------
# F	Min Frames	Frames < 10	Max Frames	Page Faults
#-------------------------------------------------------------------
# 5	1		87		30		1324
# 10	1		10		30		104
# 15	1		10		30		30
# 20	1		10		30		30
# 25	1		10		30		30
#-------------------------------------------------------------------
# The number of page faults decreases significantly with the
#  increase in the value of 'F' and becomes constant after a certain 
#  value.
#-------------------------------------------------------------------
def pff(attributes):
	pageFaultCount = 0
	fValue = 10
	tValue = 0
	minCount = 10000
	maxCount = 0
	tenCount = 0
	dictionary = {}
	for index in attributes:						# Pick Page from the List of Sequencial Page References
		key = index
		value = 1			
		if index not in dictionary.keys():				# Page Fault Occurs
			pageFaultCount = pageFaultCount + 1			# Increament Page Fault Count
			if tValue >= fValue:					# Page Fault Occurs and T >= F
				for num in dictionary.keys():			# Remove Pages with Use Bit = 0
					if dictionary[num] == 0:
						del dictionary[num]
				for num in dictionary.keys():			# Set Use Bits of Remaining Pages to 0
					dictionary[num] = 0
				dictionary[key] = value				# Add New Page to Set
				tValue = 0
			else:							# Page Fault Occurs and T < F
				dictionary[key] = value				# Add New Page to Set
				tValue = 0
		else:								# Page Found in Memory. No Page Fault Occurs
			dictionary[key] = value					# Add New Page to Set
			tValue = tValue + 1
		if len(dictionary) < minCount:					# Count Minimum Number of Frames Allocated
			minCount = len(dictionary)
		if len(dictionary) < 10:					# Count Number of Times less than 10 Frames are Allocated
			tenCount = tenCount + 1
		if len(dictionary) > maxCount:					# Count Maximum Number of Frames Allocated
			maxCount = len(dictionary)
	print "\n----------------------------------------------"
	print "Result: PFF (Parameters: F %d)" %fValue
	print "----------------------------------------------"
	print "Minimum Number of Frames Allocated : %d" %minCount
	print "Frequency of less than 10 Frames   : %d" %tenCount
	print "Maximum Number of Frames Allocated : %d" %maxCount
	print "Total Number of Page Faults in PFF : %d" %pageFaultCount
	print "----------------------------------------------\n"

# Function to Implement VSWS
#-----------------------------------------------------------------------------------
# Sample Outputs:
# 	Number of Pages the Process Occupies	: 30
#	Total Number of Page References		: 10000
#-----------------------------------------------------------------------------------
# M	L	Q	Min Frames	Frames < 10	Max Frames	Page Faults
#-----------------------------------------------------------------------------------
# 5	15	10	1		25		28		3921
# 10	20	15	1		18		30		369
# 15	25	20	1		15		30		305
# 20	30	25	1		10		30		79
# 25	35	30	1		10		30		39
#-----------------------------------------------------------------------------------
# The number of page faults decreases significantly with the increase in the values
#  of 'M', 'L' and 'Q'.
#-----------------------------------------------------------------------------------
def vsws(attributes):
	pageFaultCount = 0
	pageFaults = 0
	mValue = 20
	lValue = 30
	qValue = 25
	tValue = 0
	minCount = 10000
	maxCount = 0
	tenCount = 0
	dictionary = {}
	for index in attributes:						# Pick Page from the List of Sequencial Page References
		key = index
		value = 1			
		if index not in dictionary.keys():				# Page Fault Occurs
			pageFaultCount = pageFaultCount + 1
			pageFaults = pageFaults + 1
			if tValue >= lValue:					# If T >= L
				for num in dictionary.keys():
					if dictionary[num] == 0:		# Remove Pages with Use Bit = 0
						del dictionary[num]
				for num in dictionary.keys():			# Set Use Bits of Remaining Pages to 0
					dictionary[num] = 0
				dictionary[key] = value				# Add New Page to Set
				tValue = 0
				pageFaults = 0
			else:							# If T < L
				if pageFaults > qValue and tValue >= mValue: 	# If Virtual Time since Last Sampling >= M and pageFaults > Q
					for num in dictionary.keys():		# Remove Pages with Use Bit = 0
						if dictionary[num] == 0:
							del dictionary[num]
					for num in dictionary.keys():		# Set Use Bits of Remaining Pages to 0
						dictionary[num] = 0
					dictionary[key] = value			# Add New Page to Set			
					tValue = tValue + 1
					pageFaults = 0
				else:						# If Virtual Time since Last Sampling < M	
					dictionary[key] = value			# Add New Page to Set
					tValue = tValue + 1
		else:								# Page Found in Memory. No Page Fault Occurs
			dictionary[key] = value					# Add New Page to Set
			tValue = tValue + 1
		if len(dictionary) < minCount:					# Count Minimum Number of Frames Allocated
			minCount = len(dictionary)
		if len(dictionary) < 10:					# Count Number of Times less than 10 Frames are Allocated
			tenCount = tenCount + 1
		if len (dictionary) > maxCount:					# Count Maximum Number of Frames Allocated
			maxCount = len (dictionary)
	print "\n----------------------------------------------"
	print "Result: VSWS (Parameters: M %d, L %d, Q %d)" %(mValue, lValue, qValue)
	print "----------------------------------------------"
	print "Minimum Number of Frames Allocated : %d" %minCount
	print "Frequency of less than 10 Frames   : %d" %tenCount
	print "Maximum Number of Frames Allocated : %d" %maxCount
	print "Total Number of Page Faults in VSWS: %d" %pageFaultCount
	print "----------------------------------------------\n"

# Main Function
def main ():
	print "\n\n----------------------------------------------"
	print "Page Replacement Algorithms: PFF & VSWS"
	print "----------------------------------------------"
	fileName = sys. argv[1]
	attributes = readFile (fileName)
	print "Input File Name                     : %s" %fileName
	print "Number of Pages the Process Occupies: %s" %attributes[0]
	print "----------------------------------------------\n"
	attributes.remove (attributes[0])
	pff (attributes)
	vsws (attributes)	

if __name__ == "__main__":
	main ()
```

## Reference

- https://github.com/WeichenXu/Mem_Resident_Size/blob/master/mem_manage.c
- https://github.com/rahulkalpsts107/paging-algorithm
- https://github.com/rupanta2009/Operating-Systems-Memory-Management-PFF-VSWS-Algorithms-