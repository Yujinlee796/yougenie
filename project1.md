
## Project 1 report
제출자 : 2018-15485 재료공학부 이유진   
제출일 : 2020-05-18
### 1. code 설명   
따로 파일이나 함수를  추가하지 않고 주어진 함수로 구현하였다. 아래는 sort.cpp 내의 함수 code에 대한 설명이다.  

- swap 함수 : array 배열을 교환하는 주어진 swap 함수를 그대로 사용하였다.
```c++
void swap(int *a, int *b)
{
  int temp = (*a);
  (*a) = (*b);
  (*b) = temp;
}
```

1. Bubble Sort 함수 
```c++
void bubbleSort(int *arr, int arrLength)
{
    //level : 1개의 index를 정렬하는 과정을 의미 (강의에서 사용된 level과 동일한 의미)
    for(int level{1}; level < arrLength; level++)    
    {   
        ////iter은 맨처음 index부터 시작 각 level 마다 오른쪽에서부터 level개의 숫자는 비교할 필요 없음.
        for (int iter{0}; iter < arrLength - level; iter++) 
        {
            // 왼쪽 수가 오른쪽 수보다 작으면 swap 
            if(*(arr+iter) > *(arr+iter+1)) swap(&arr[iter],&arr[iter+1]);  
            // 그렇지 않으면 다음 iter 진행
            else continue;                                                  
        }
    }
}
```   

2. SelectionSort.cpp
```c++
void selectionSort(int *arr, int arrLength)
{
    // sortIndex : 현재 정렬하려는 index
 
    for(int sortIndex{0}; sortIndex < arrLength - 1; sortIndex++)
    {
        int* min = &arr[sortIndex]; //min : 정렬되지 않은 array 중 최솟값을 가리키는 변수
        
        //최솟값 찾는 과정
        for (int iter{sortIndex}; iter < arrLength ; iter++)
        {
            if (*min > arr[iter]) min = &(arr[iter]);
        }
  
        swap(&arr[sortIndex],min);        //최솟값을 sortIndex에 정렬
    }
}
```   

3. Insertion.cpp
```c++
void insertionSort(int *arr, int arrLength)
{
    //sortIndex : 삽입 정렬을 하려는 index
    for(int sortIndex{1}; sortIndex < arrLength ; sortIndex++)
    {
        int temp = arr[sortIndex];   //temp : 현재 삽입하려는 값을 임시로 저장

                                     //삽입
        int insert = sortIndex;      //Insert : 삽입해야할 index를 의미하는 변수
        while(arr[insert-1] > temp)  //삽입하려는 값과 삽입하려는 값 이전 index의 값 비교
        {
            arr[insert] = arr[insert-1];  //index가 더 크다면 오른쪽으로 1칸씩 이동
            insert--;                     //한칸씩 앞으로 당기면서 비교
        }

        arr[insert] = temp;          //삽입 위치에 arr[sortIndex] 값을 넣어줌.
    }
}
```   

4. Merge Sort    
 * mergeSort.cpp
```c++
void mergeSort(int*arr,int left,int right,int num)
{
  if (left < right)  //크기가 1인 배열이 되면 더이상 함수를 호출하지 않음
  {
    int mid = (left+right)/2;
    mergeSort(arr,left,mid,num);
    mergeSort(arr,mid+1,right,num);
    merge(arr,left,mid,right,num);
  }
}
```   

 * merge.cpp
```c++
void merge(int *arr, int left, int mid, int right, int num)
{
    int* tempArr = new int[num]; //합병 결과를 저장할 임시배열 만들기

    int i = left;    //배열 1의 시작 index
    int j = mid + 1; //배열 2의 시작 index
    int k = left;    //임시배열의 시작 index

    while( i <= mid && j <= right) //두 배열 중 하나라도 끝에 도달할때까지 반복
    {
        if(arr[i] <= arr[j])       //배열 1이 배열 2보다 작으면
            tempArr[k] = arr[i++];
       else                        //배열 2가 배열 1보다 작으면
            tempArr[k] = arr[j++];
        k++;
    }    

    //while문 벗어난 후 남은 수 배열
    if(i > mid) //배열 1이 먼저 끝난 경우
    {
        for(;k<=right;k++){
            tempArr[k] = arr[j];  // 배열 2의 남은 수를 tempArr에 저장
            j++;
        }
    }
   else        //배열 2가 먼저 끝난 경우
   { 
        for(;k<=right;k++){
            tempArr[k] = arr[i]; // 배열 1의 남은 수를 tempArr에 저장
            i++;
        }
   }
                               
   for(int iter{left}; iter <= right; iter++){     //arr에 tempArr 복사
    arr[iter] = tempArr[iter];
   }

  delete[] tempArr; //임시배열 삭제
}
```

### 2.Graph
windows terminal을 이용하여 array size를 5,000부터 100,000까지 5,000단위로 시행하였다. array의 배열에 따라 수행시간에 차이를 보이기 때문에 일반적인 경향성을 반영하기 위해서 5번 시행한뒤 평균값을 구했다. 다음 그래프는 그 평균값을 나타낸 그래프이다. (raw data는 sorting result.xlsx 에 정리하여 etl에 첨부)   

![sortingtimegraph](./sortingtimegraph.PNG)   

 수행 시간은 bubble > selection > insertion 순서로 나타났다. graph 개형으로 보아 Bubble Sort, Selection Sort, Insertion Sort, Merge Sort의 수행시간은 ~O(n^2), 즉 데이터 개수의 제곱에 비례함을 확인할 수 있었다.    
 merge sort의 경우 array size가 100,000개일 때도 수행시간의 평균이 0.037502로, 준수한 sorting 속도를 보였다. 위 그래프로는 merge sort의 경향성을 파악하기가 어렵기 때문에 따로 그래프를 나타내었다.   
 
 
 ![mergesorttime](./mergesorttime.PNG)   
 
 그래프 값의 편차는 있었지만 이는 mergesort 수행시간이 매우 짧기 때문으로 보인다. 또한 위의 O(n^2)의 다른 그래프 개형과 비교했을때, 확연히 완만한 경향을 확인할 수 있어 O(nlogn)을 따른다는 것을 확인할 수 있었다.   
 
 더 큰 data개수에 대한 merge sort 결과를 그래프로 나타내보았다.   
 
 ![bigmergesort](./bigmergesort.PNG)   
 
 100000,30000,50000,100000,300000,500000,1000000개의 arraysize로 mergesort를 시행해보았다. 위의 graph보다 더 확실하게 O(nlogn)을 따른다는 것을 조금 더 확실하게 확인할 수 있었다. 
 
