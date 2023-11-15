


### **in** AFL_FUZZ.main().while

1. cull_queue()

2. show_stats()

3. fuzz_one(use_argv) 
> 가장 중요한 함수라고 생각됨

4. (maybe?) sync_fuzzers(use_argv)


## 

### celibrate_case 
> 새로운 경로 탐색

## fuzz_one(use_argv)
### 피드백?

#### calculate_score(queue_cur)
> 스코어를 반환함

1. exec_us
> Execution time (us)
> 실행 시간이 짧다 -> 스코어 점수를 높게 쳐준다

2. bitmap_size
> Number of bits set in bitmap
> 비트맵 사이즈가 크다 -> 스코어 점수를 높게 쳐준다

3. depth
> 깊을수록 스코어 업

4. 핸디캡?

> 해당 스코어를 어디서 사용?

##

### mutate?

1. bitflip 1/1
> common_fuzz_stuff(argv, out_buf, len)
> maybe_add_auto()

2. bitflip 2/1
> common_fuzz_stuff(argv, out_buf, len)

3. bitflip 4/1
> common_fuzz_stuff(argv, out_buf, len)

** Walking Byte **
4. bitflip 8/8
> common_fuzz_stuff(argv, out_buf, len);

5. bitflip 16/8

6. bitflip 32/8

** arithmetics -> 산술 연산 -> add, sub, OR, AND, XOR **
1. arith 8/8
> could_be_bitflip(r) ** 아마도 0xFF, 0xFFFF? 일 경우 오버 플로가 발생할 수 있어서? **
> common_fuzz_stuff()

2. arith 16/8

3. arith 32/8

** Interesting values 뮤테이트 **
1. interest 8/8

2. interest 16/8

3. interest 32/8


#### interesting 한 값들 
```c
#define INTERESTING_8 \
  -128,          /* Overflow signed 8-bit when decremented  */ \
  -1,            /*                                         */ \
   0,            /*                                         */ \
   1,            /*                                         */ \
   16,           /* One-off with common buffer size         */ \
   32,           /* One-off with common buffer size         */ \
   64,           /* One-off with common buffer size         */ \
   100,          /* One-off with common buffer size         */ \
   127           /* Overflow signed 8-bit when incremented  */

#define INTERESTING_16 \
  -32768,        /* Overflow signed 16-bit when decremented */ \
  -129,          /* Overflow signed 8-bit                   */ \
   128,          /* Overflow signed 8-bit                   */ \
   255,          /* Overflow unsig 8-bit when incremented   */ \
   256,          /* Overflow unsig 8-bit                    */ \
   512,          /* One-off with common buffer size         */ \
   1000,         /* One-off with common buffer size         */ \
   1024,         /* One-off with common buffer size         */ \
   4096,         /* One-off with common buffer size         */ \
   32767         /* Overflow signed 16-bit when incremented */

#define INTERESTING_32 \
  -2147483648LL, /* Overflow signed 32-bit when decremented */ \
  -100663046,    /* Large negative number (endian-agnostic) */ \
  -32769,        /* Overflow signed 16-bit                  */ \
   32768,        /* Overflow signed 16-bit                  */ \
   65535,        /* Overflow unsig 16-bit when incremented  */ \
   65536,        /* Overflow unsig 16 bit                   */ \
   100663045,    /* Large positive number (endian-agnostic) */ \
   2147483647    /* Overflow signed 32-bit when incremented */
```
```c
static s8  interesting_8[]  = { INTERESTING_8 };
static s16 interesting_16[] = { INTERESTING_8, INTERESTING_16 };
static s32 interesting_32[] = { INTERESTING_8, INTERESTING_16, INTERESTING_32 };
```


** Dictionary ** 
1. user extras (over)

2. user extras (insert)

3. auto extras (over)




#### common_fuzz_stuff

write_to_testcase(out_buf, len)

run_target(argv, exec_tmout) 
> FAULT_CRASH > 2
> FAULT_TMOUT > 1
> FAULT_NONE > 0

save_if_interesting(argv, out_buf, len, fault)
    calibrate_case()


## save_if_interesting(argv, out_buf, len, fault)
> C 옵션 -> crash_mode

hnb = has_new_bits(virgin_bits)

  res = calibrate_case(argv, queue_top, mem, queue_cycle - 1, 0);

#### has_new_bits(virgin_bits)
trace_bits > 코드 커버리지 정보
    > run_target에서 exec 하기 전에 0으로 

> current -> 현재 코드 커버리지
> virgin_bits -> 이전 코드 커버리지 정보(~current) + 안바뀐 코드커버리지 정보(0xFF)


```c 
EXP_ST void setup_shm(void) {

  u8* shm_str;

  if (!in_bitmap) memset(virgin_bits, 255, MAP_SIZE);

```

```c
    add_to_queue(fn, len, 0);

    if (hnb == 2) {
      queue_top->has_new_cov = 1;
      queued_with_cov++;
    }
```

#### add_to_queue(fn, len, 0);
```c
    fn = alloc_printf("%s/queue/id:%06u,%s", out_dir, queued_paths,
                      describe_op(hnb));
```
describe_op
```c
if (hnb == 2) strcat(ret, ",+cov");
```


#### fault = run_target(argv, exec_tmout);

1. memset(trace_bits, 0, MAP_SIZE);


#### calibrate_case
> for문 돌려서 3, 8번 반복함 

> 경로가 더 나오나 확인~

write_to_testcase
run_target

q에 채운다  ---->...





# 코드 흐름 파악

main()
    get args
    `/afl/afl-fuzz -i ./WICHR/work/initial_seeds -o ./WICHR/work -m 8G -M fuzzer-master -x ./WICHR/work/dict.txt -t 5000+ -- /usr/local/bin/php-cgi ''`

        -i : input dir
            in_dir
        
        -o : output dir
            out_dir
        
        -m : memory limit
            mem_limit
            mem_limit_given = 1

        -M : master sync ID
            sync_id 
            c
            force_deterministic = 1
        
        -x : dictionary
            extras_dir 
        
        -t : timeout
            timeout_given = 1 or 2
            

    setup_signal_handlers():
        handle_stop_sig
            SIGHUP, SIGINT, SIGTERM
        
        handle_timeout
            SIGALRM
        
        handle_resize
            SIGWINCH

        handle_skipreq;
            SIGUSR1
        
        SIG_IGN ( ignore )
            SIGTSTP
            SIGPIPE

    check_asan_opts():
        getenv ("ASAN_OPTIONS")
        getenv ("MSAN_OPTIONS")

    [if]fix_up_sync():
        파일명에서 알파벳, -, _가 아니면 에러
        32바이트보다 길면 에러
        out_dir + sync_id = *x
        out_dir = *x

        
    save_cmdline(argc, argv):
        buf, orig_cmdline 에 현재 명령 줄을 담음


    fix_up_bannder(argv[optind]):

    check_if_tty():
        터미널 확인 -> 사이즈 확인
        
    get_core_count():
        get /proc/stat
        cpu 코어 카운트 추출

    check_crash_handling():
        process /proc/sys/kernel/core_pattern

    check_cpu_governor():
        cpu 스케쥴러 관리 ?

    setup_post():
        퍼징 끝난 후 실행되는 사용자 지정 후처리 작업

    setup_shm():
        공유 메모리 
        virgin_bits, virgin_tmout, virgin_crash -> 0xff 0xff...

        shm_id = shmget(IPC_PRIVATE...)

        atexit(remove_shm)

        "__AFL_SHM_ID" 에 shm_id 값을 넣음 (%d)

        trace_bits = shmat(shm_id)
        

    init_count_class16():

    setup_dirs_fds():
        다양한 폴더 생성 

    read_testcases():
        in_dir -> -i 매개변수 -> '-i ./WICHR/work/initial_seeds'

        nl 변수에 저장
        예시 
            html_response_page=back.asp&html_response_return_page=st_ipv6.asp&html_response_message=

        u8* fn = alloc_printf("%s/%s", in_dir, nl[i]->d_name);

        add_to_queue(fn, st.st_size, 0 or 1[x]):
            fname -> 경로 + 파일명 (seed-0)

            첫 seed가 queue + queue_top

            queued_paths++
            pending_not_fuzzed++



    load_auto():
        -M + .state + auto_extras / auto_0...
    

    pivot_inputs():
        q = queue 
        
        while(q){

            *rsl = 'seed-0'의 [0]

            최종적으로 초기 값으로 
            queue 폴더에 

            id:000000,orig:seed_0 이라는 파일을 만듬
            link_or_copy(q->fname, nfn);
            // 
        }


    [if]load_extras():
        dict.txt 파일을 불러옴 -x ''
        
        파일 이기에..
        load_extras_file():
            extras
            extras_cnt

        qsort(extras, )

        -> -x 로 받은 파일을 읽어서 extras에 채운다


    detect_file_args():
        @@?

    setup_stdio_file():
        out_dir/.cur_input 파일 만들고 
        out_fd = 

    check_binary():
        바이너리 체크? "PATH"?

    get_cur_time():

    use_argv = argv + optind;
        use_argv => -i -f 이거 다 제외한거..
        '/afl/afl-fuzz -i ./WICHR/work/initial_seeds -o ./WICHR/work -m 8G -M fuzzer-master -x ./WICHR/work/dict.txt -t 5000+ -- /usr/local/bin/php-cgi '''
        ->/usr/local/bin/php-cgi, ''


    
    perform_dry_run():  
        
        

    cull_queue()

    show_init_stats()

    find_start_position()

    write_stats_file()

    save_auto()

    while(1){
        cull_queue()

        fuzz_one() -> mutate

        [if]sync_fuzzers()

        stop_soon -> CTRL + C


        query_cur = query_cur->next;
        current_entry++;
    }

    


