```c
const char* svuln = randomSelection(vulnsMap);
char buffer[1 * 1024 * 1024];
mutatedSeed = ck_alloc(1*1024*1024);
FILE* qfn = fopen(queue_search->fname,"rb");
int buf_size = fread(buffer, 1, (1 * 1024 * 1024), qfn);

mutate(mutatedSeed, svuln, buffer, buf_size);
```


## int mutate(char* ret, const char* vuln, char* seed, int length)


\x00 -> post 면 \x00 로 데이터 영역이 나뉘어져 있음
get = strdup(buffer);
post = strdup(buffer);

strdup > 문자열 복사
strtok > 구분자로 사용하여 토큰을 나눔

getValue
postValue

모든 Value 들을 mutate?