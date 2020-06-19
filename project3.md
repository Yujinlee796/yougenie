```c++
//문자별 빈도수를 계산하는 함수
void HuffmanCode::calcFreq(string sourcefile)
{
  ifstream file(sourcefile);
  char ch;

  while(file.get(ch)) 
  {
   freqs[ch]++;    //한글자씩 읽고 key에 해당하는 value++
  }
}

void HuffmanCode::buildHeap()
{
  //data를 priority queue에 넣기
  map<char,int>::iterator iter;
  for(iter=freqs.begin(); iter!=freqs.end(); iter++) {
   Node* temp = new Node();
   temp->ch = iter->first;
   temp->freq = iter->second;
   pq.push(temp);
  }
}

void HuffmanCode::buildHuffmanTree(string source_file)
{
  //문자별 빈도수 계산
  calcFreq(source_file);
  //문자별 빈도수를 priority queue에 넣음
  buildHeap();

  if(freqs.empty()) return;

  //tree 생성
  for(int i=0; i < freqs.size()-1; i++) {
   Node* z = new Node();
   z->left = pq.top();
   pq.pop();
   z->right = pq.top();
   pq.pop();
   //빈도수를 합한 new node 다시 priority queue에  넣어줌
   z->freq = z->right->freq + z->left->freq;
   pq.push(z);
  }

}

void HuffmanCode::makeHuffmanCodeTable(Node *node,string str)
{ 
  //왼쪽 node로 이동할때는 0 가중치
  if(node->left != nullptr) {
   str = str + "0";
   makeHuffmanCodeTable(node->left, str);
  }
  //오른쪽 node로 이동할 때는 1 가중치
  if(node->right != nullptr) {
   str = str + "1";
   makeHuffmanCodeTable(node->right, str);
  }
  //leaf node에 도달하면 huffmanCodeTable에 저장
  if((node->left == nullptr) && (node->right == nullptr)) {
   huffmanCodeTable[node->ch] = str;
  }
}


void HuffmanCode::printHuffmanTable()
{ 
  //1개씩 huffmanCodeTable 생성
  while(!pq.empty()) {
   Node* temp = pq.top();
   makeHuffmanCodeTable(temp,"");
   pq.pop();
  }

  cout << "Huffman Code Table" << endl;
  map<char,string>::iterator iter;
         
  //huffmanCodeTable 출력
  for (iter = huffmanCodeTable.begin(); iter != huffmanCodeTable.end(); iter++) {
   char ch = iter->first;
   if(ch == ' ') cout << "ws" << " : " << iter->second << endl;
   else if(ch == '\n') cout << "nl" << " : " << iter->second << endl;
   else cout << iter->first << " : " << iter->second << endl;
  }

  cout << endl;
}


void HuffmanCode::encode(string input_file, string encoded_file)
{
  ifstream inf{ input_file };
  ofstream outf{ encoded_file };
  char ch;
  map<char,string>::iterator iter;

  //file의 문자를 1개씩 읽고 huffmanCodeTable의 huffman code를 출력
  while(inf.get(ch)) {
   iter = huffmanCodeTable.find(ch);
   outf << iter->second << '\n';
  }

  inf.close();
  outf.close();
}


void HuffmanCode::decode(string encoded_file, string decoded_file)
{ 
  ifstream inf{encoded_file};
  ofstream outf{decoded_file};
  string str;

  while(getline(inf,str)) {
   map<char,string>::iterator iter;
          
   //value로 key를 찾기 위해서 처음부터 끝까지 검색
   for(iter = huffmanCodeTable.begin(); iter!=huffmanCodeTable.end(); iter++) { 
    if(iter->second == str) outf << iter->first; 
    else continue;
   }
  }

  inf.close();
  outf.close();
}
```

```c++
int charToIndex(char ch)
{
  //대문자변환
  if ((ch >= 65) && (ch <= 90)) return ch-65;
  //소문자변환
  else if ((ch >= 97) && (ch <= 122)) return ch-51;
  //특수문자 변환
  else if (ch == 44) return 52; //','
  else if (ch == 46) return 53; //'.'
  else if (ch == 33) return 54; //'!'
  else if (ch == 63) return 55; //'?'
  else if (ch == 32) return 56; //' '
  else if (ch == 10) return 57; //'\n'
  
}

char indextoChar(int index)
{ 
  //대문자변환
  if (index<26) return index+65;
  //소문자변환
  else if ((index >= 26) && (index < 52)) return index+97;
  //특수문자 변환
  else if (index == 52) return 44;  //','
  else if (index == 53) return 46;  //'.'
  else if (index == 54) return 33;  //'!'
  else if (index == 55) return 63;  //'?'
  else if (index == 56) return 32;  //' '
  else if (index == 57) return 10;  //'\n'
}

///////////////////////
//   MinHeap Class   //
///////////////////////
Node* MinHeap::top()
{
 return heapArray[0];
}

void MinHeap::swap(Node *x, Node *y) 
{
 Node* temp = x;
 x = y;
 y = temp;
}

//pop 후 heap 정렬
void MinHeap::heapify(int i)
{
  int point = i;

  int lc = 2*point + 1;
  int rc = lc + 1;
  int k = point;
  if( lc<size && heapArray[lc]->freq < heapArray[k]->freq) k = lc;
  if( rc<size && heapArray[rc]->freq < heapArray[k]->freq ) k = rc;
  if( k!=point ) {
   swap(heapArray[k],heapArray[point]);
   heapify(k);
  }
}

void MinHeap::push(Node *node)
{
  heapArray[size] = node;
  
  if( size > 0 ) {
   int i = size;
   int p;

   while(true) {
    if(i%2 == 0) p = i/2 - 1;
    else if (i%2 == 1) p = (i+1)/2 - 1;
    
    //root에 도달하면 break
    if(p == 0) break;

    //배열이 올바르면 break
    if(heapArray[p]->freq < heapArray[i]->freq) break;
    
    //아니라면 swap
    swap(heapArray[p],heapArray[i]);
    }
   }
  size++;
}

void MinHeap::pop()
{
  heapArray[0] = heapArray[size];
  size--;
  heapify(0);
}
 
///////////////////////////
//   HuffmanCode Class   //
///////////////////////////

void HuffmanCode::calcFreq(string source_file)
{
  ifstream file(source_file);
  char ch;
  while(file.get(ch)) {
   for(int i=0; i < 58; i++) {  
    freqs[charToIndex(ch)]++;
   }
 }
}

void HuffmanCode::buildHeap()
{ 
  //Minheap에 넣어줌
  for(int i=0; i < 58; i++){
   Node* alphabet = new Node();
   alphabet->ch = indextoChar(i);
   alphabet->freq = freqs[i];
   heap.push(alphabet);
  }
}


void HuffmanCode::buildHuffmanTree(string source_file)
{
  calcFreq(source_file);
  buildHeap();

  while(heap.getSize() > 2) 
  {
   Node* sum = new Node();
   sum->freq = heap.top()->freq;
   sum->left = heap.top();
   heap.pop();

   sum->freq = sum->freq + heap.top()->freq;
   sum->right = heap.top();
   heap.pop();

   cout << sum->freq << endl;
   heap.push(sum);
  }
}

void HuffmanCode::makeHuffmanCodeTable(Node* node, string str)
{
 if(node->left != nullptr) {
  str = str + "0";
  makeHuffmanCodeTable(node->left,str);
 }

 if(node->right != nullptr) {
  str = str + "1";
  makeHuffmanCodeTable(node->right,str);
 }

 if((node->left == nullptr) && (node->right == nullptr)) {
  huffmanCodeTable[charToIndex(node->ch)] = str;
 }
 cout << "Table end" << endl;
}

void HuffmanCode::printHuffmanTable()
{
  while ( heap.getSize() != 0 ) {
   Node* temp = heap.top();
   makeHuffmanCodeTable(temp,"");
   heap.pop();
  }

  cout << "Huffman Code Table" << endl;
  for (int i=0; i<58; i++) {
   if(i == 56) cout << "ws" << " : " << huffmanCodeTable[i] << endl;
   else if(i==57) cout << "nl" << " : " << huffmanCodeTable[i] << endl;
   else cout << indextoChar(i) << " : " << huffmanCodeTable[i] << endl;
  }

  cout << endl;
}


void HuffmanCode::encode(string input_file, string encoded_file)
{
  ifstream inf{ input_file };
  ofstream outf{ encoded_file }; 
  char ch;
  
  while(inf.get(ch)) {
   outf << huffmanCodeTable[charToIndex(ch)] << '\n';
  }

  inf.close();
  outf.close();
}


void HuffmanCode::decode(string encoded_file, string decoded_file)
{
  ifstream inf{encoded_file};
  ofstream outf{decoded_file};
  string str;

  while(getline(inf,str)) {
   for( int i=0; i <58; i++) {
    if( str == huffmanCodeTable[i] ){
     outf << indextoChar(i);
     break;
    }
    else continue;
   }
  }
 
  inf.close();
  outf.close();
}

```
