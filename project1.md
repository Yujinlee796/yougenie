# Project 1 report
제출자 : 2018-15485 재료공학부 이유진   
제출일 : 2020-05-18
## 1. code 설명   
*swap 함수
```c++

1. bubbleSort.cpp
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
  
        //최솟값을 sortIndex에 정렬
        swap(&arr[sortIndex],min);
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
        //temp : 현재 삽입하려는 값을 임시로 저장
        int temp = arr[sortIndex];

        //삽입
        int insert = sortIndex;      //Insert : 삽입해야할 index를 의미하는 변수
        while(arr[insert-1] > temp)  //삽입하려는 값과 삽입하려는 값 이전 index의 값 비교
        {
            arr[insert] = arr[insert-1];  //index가 더 크다면 오른쪽으로 1칸씩 이동
            insert--;                     //한칸씩 앞으로 당기면서 비교
        }

        //삽입 위치에 arr[sortIndex] 값을 넣어줌.
        arr[insert] = temp;
    }
}
```   

4.Merge Sort    
*mergeSort.cpp
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

*merge.cpp
```c++
void merge(int *arr, int left, int mid, int right, int num)
{
    int* tempArr = new int[num]; //합병 결과 저장할 임시배열 만들기

    int i = left; //배열 1의 시작 index
    int j = mid + 1; //배열 2의 시작 index
    int k = left; //임시배열의 시작 index

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
            tempArr[k] = arr[j];  // 남은 배열 2의 수를 tempArr에 저장
            j++;
        }
    }
   else        //배열 2가 먼저 끝난 경우
   { 
        for(;k<=right;k++){
            tempArr[k] = arr[i]; // 남은 배열 1의 수를 tempArr에 저장
            i++;
        }
   }

   //arr에 tempArr 복사
   for(int iter{left}; iter <= right; iter++){
    arr[iter] = tempArr[iter];
   }

  delete[] tempArr; //임시배열 삭제
}
```

## 2.Graph
------------------
