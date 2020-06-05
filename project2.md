```c++
/********************** FIFO Read *****************************/
#ifdef FIFO
data_t Cache::read_data(addr_t address)
{ 
 globalClock += cache_access_time;

 //hit 여부 조사 -> hit이면 해당값 return
 for(int i=0; i<size; i++)
 {
  if(cache_cells[i].address == address) 
  {
   return cache_cells[i].value;
  }
 }

 // miss
 // 1. Check if there is invalid cell
 for(int i=0; i<size; i++)
 {
  if(cache_cells[i].valid==false)                   //invalid cell에 load
  {
   cache_cells[i].address = address;              
   cache_cells[i].value = mem->read_data(address);
   cache_cells[i].valid = true;
   return cache_cells[i].value;                  
  }
  else continue;
 }

 // 2. if not, evict a cell and replace
#ifdef WRITE_BACK
  //check dirty before evict
 if(cache_cells[fifo_head_idx].dirty == true)     
 {
  mem->write_data(fifo_head_idx,cache_cells[fifo_head_idx].value);
  cache_cells[fifo_head_idx].dirty = false;
 }
#endif

 cache_cells[fifo_head_idx].address = address;
 cache_cells[fifo_head_idx].value = mem->read_data(address);
 data_t temp = cache_cells[fifo_head_idx].value;

 if(fifo_head_idx == size-1) fifo_head_idx = 0;
 else fifo_head_idx = fifo_head_idx + 1;

 return temp;
}
#endif  //end FIFO

/*************************************************************/
```

```c++
/********************** LRU Read *****************************/
#ifdef LRU
data_t Cache::read_data(addr_t address)
{ 
 globalClock += cache_access_time;
 
 // Cache read hit? : Check if target is in cache
 for(int i=0; i<size; i++)
 {
  // LRU policy에 맞게 sort
  if(cache_cells[i].address == address)
  {
   data_t temp = cache_cells[i].value;
   for(int idx=i; idx<size-1; idx++)
   {
    cache_cells[idx] = cache_cells[idx+1];
   }
   cache_cells[size-1].address = address;
   cache_cells[size-1].value = temp;
   return temp;
  }
 }
    
 // Cache read miss : 
 // 1. Check if there is invalid cell
 // LRU policy는 0이 가장 늦게 채워지므로 큰 index부터 check

 for(int i=size-1; i>=0; i--)
 {
  if(cache_cells[i].valid == false)
  {
   // sort
   for(int idx=i; idx<size-1; idx++)
   {
    cache_cells[idx] = cache_cells[idx+1];
   }
   cache_cells[size-1].address = address;
   cache_cells[size-1].value = mem->read_data(address);
   cache_cells[size-1].valid = true;
   return cache_cells[size-1].value;
  }
  else continue;
 }

 // if not, evict a cell and replace
 // evict하기 전에 write back에 대한 처리를 해줌
#ifdef WRITE_BACK
 if(cache_cells[0].dirty == true)
 {
  mem->write_data(cache_cells[0].address,cache_cells[0].value);
  cache_cells[0].dirty = false;
 }
#endif

 // 앞에서 invalid cell이 존재하는 경우를 처리했으므로 cache 전체가 채워져 있을 것임
 for(int idx=0; idx<size-1; idx++)
 {
  cache_cells[idx] = cache_cells[idx+1];
 }
 cache_cells[size-1].address = address;
 cache_cells[size-1].value = mem->read_data(address);
 return cache_cells[size-1].value;

}
#endif  //end LRU
```

```c++
/********************** FIFO write ***************************/
#ifdef FIFO
void Cache::write_data(addr_t address, data_t value)
{
 globalClock += cache_access_time;

 // Cache write hit? : Check if target is in cache      
#ifdef WRITE_THROUGH
 //hit인 경우, cache와 memory 둘다 update
 for(int i=0;i<size;i++)
 {
  if(cache_cells[i].address == address)
  {
   cache_cells[i].value = value;
   mem->write_data(address,value);
   return;
  }
  else continue;
 }  
#endif

#ifdef WRITE_BACK
 //hit인 경우, cache만 update후 dirty값 true로 변경
 for(int i=0;i<size;i++)
 {
  if(cache_cells[i].address == address)
  {
   cache_cells[i].value = value;
   cache_cells[i].dirty = true;
   return;
  }
  else continue;
 }
#endif


 // Cache write miss : 
 // 1. Check if there is invalid cell
 // 2. if not, evict a cell and replace according to FIFO

#ifdef WRITE_THROUGH
 //miss인 경우, memory만 update
 mem->write_data(address,value);

#endif

#ifdef WRITE_BACK
 //miss인 경우, memory update 후 cache에도 배정
 for(int i=0;i<size;i++)
 {
  //cache에 invalid cell이 있으면, 넣어준다.
  if(cache_cells[i].valid == false)
  {
   cache_cells[i].address = address;
   cache_cells[i].value = value;
   cache_cells[i].valid = true;
   mem->write_data(address,value);
   return;
  }
  else continue;
 }
  
 //cache가 꽉 찬 경우 : evict and replace
 //evict 전 dirty 값 check 후 memory update
 if(cache_cells[fifo_head_idx].dirty == true) 
 {
  mem->write_data(cache_cells[fifo_head_idx].address,cache_cells[fifo_head_idx].value);
  cache_cells[fifo_head_idx].dirty = false;
 }

 //evict된 자리에 write
 cache_cells[fifo_head_idx].address = address;
 cache_cells[fifo_head_idx].value = value;
  
 //fifo head 정렬
 if(fifo_head_idx == size-1) fifo_head_idx = 0;
 else fifo_head_idx = fifo_head_idx + 1;

#endif  //end write back
}
#endif  //end FIFO
/*************************************************************/
```

```c++
/********************** LRU write ****************************/
#ifdef LRU
void Cache::write_data(addr_t address, data_t value)
{
 globalClock += cache_access_time;

 // Cache write hit? : Check if target is in cache 
#ifdef WRITE_THROUGH
 // hit인 경우, memory와 cache에 update
 for(int i=size-1; i>=0; i--)
  {
   if(cache_cells[i].address == address)
   {
    //sort
    for(int idx=i; idx<size; idx++)
    {
     cache_cells[idx] = cache_cells[idx+1];
    }
    mem->write_data(address,value);        //memory update
    cache_cells[size-1].address = address; //cache update
    cache_cells[size-1].value = value;
    cache_cells[size-1].valid = true;
    return;
   }
   else continue;
  }
#endif

#ifdef WRITE_BACK
 // hit인 경우, cache update, dirty 값 변경
 for(int i=0; i<size; i++)
 {
  if(cache_cells[i].address == address)
  {
   for(int idx=i; idx<size-1; idx++)
   {
    cache_cells[idx] = cache_cells[idx+1];
   }
   cache_cells[size-1].address = address;
   cache_cells[size-1].value = value;
   cache_cells[size-1].valid = true;
   cache_cells[size-1].dirty = true;
   return;
  }
 }
#endif

 // Cache write miss - write through - no_allocate

#ifdef WRITE_THROUGH
 //miss인 경우, memory만 update
 mem->write_data(address,value);
#endif

 /* Cache write miss - write back - allocate
  1. Check if there is invalid cell
  2. if not, evict a cell and replace according to LRU
  */
#ifdef WRITE_BACK
  
 //invalid cell이 있을 때
 for(int i=size-1; i>=0; i--)
 {
  if(cache_cells[i].valid == false)
  {
   //memory에 할당
   mem->write_data(address,value);
   for(int idx=i; idx<size-1; idx++)
   {
    cache_cells[idx] = cache_cells[idx+1];
   }
   cache_cells[i].valid = true;       //cache_cells[i]도 채워짐
   cache_cells[size-1].address = address;
   cache_cells[size-1].value = value;
   cache_cells[size-1].valid = true;
   return;
  }
  else continue;
 }

 //invalid cell이 없을 때
 if(cache_cells[0].dirty == true)
 {
  mem->write_data(cache_cells[0].address,cache_cells[0].value);
 }

 mem->write_data(address,value);      //memory에 할당
 for(int idx=0; idx<size-1; idx++)
 {
  cache_cells[idx] = cache_cells[idx+1];
 }
 cache_cells[size-1].address = address;
 cache_cells[size-1].value = value;
 cache_cells[size-1].valid = true;
 return;
#endif  //end write_back

}
#endif  //end LRU
```
