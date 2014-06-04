### Use double bang(!!) to change a variable to 0 or 1
Useful for returning true or false based on a value > 0

    main()
    {
        unsigned long int var = 5;
        printf("%d %d %d\n",var, !var, !!var);
    }


### Use printf("%.*s",length,buf) to print a specific length from a buffer
Not very safe but useful for debugging circular buffers and the like.

    char buf[10] = "abcdefghij";
    main()
    {
        printf("%.*s\n",2,buf);
        printf("%.*s\n",4,buf);
        printf("%.*s\n",6,buf);
        printf("%.*s\n",8,buf);
        printf("%.*s\n",10,buf);
    }

### Associate strings with values

If you have some defined or enummed values sometimes you want to associate strings with them to help with debug

    enum MyErrors
    {
        EINVAL,
        ENOMEM,
        EFAULT
    };

You can init your string array using these names to make it either to correlate them

    char *err_strings[] =
    {
        [0]      = "Success",
        [EINVAL] = "Invalid argument",
        [ENOMEM] = "Not enough memory",
        [EFAULT] = "Bad address"
    };

Now you can printf("%s",err_string[err]) and easily see what's going on

Sometimes all you want to do is spit the string itself out for debug

    enum Events
    {
        IDLE,
        ENTER,
        EXIT,
        TIMER_TICK
    };

With a little preprocessor help we can make the initialization of these strings a lot easier

    const char* event_string[] =
    {
    #define EVENT_NAME(s) [s] = #s
        EVENT_NAME(IDLE),
        EVENT_NAME(ENTER),
        EVENT_NAME(EXIT),
        EVENT_NAME(TIMER_TICK)
    };

### Call a list of functions

    #include <stdarg.h>
    
    void CallThese(void (*first)(void), ...)
    {
        va_list arg_list;
        void (*fn)() = NULL;
        va_start(arg_list, first);
        (*first)();
        while ((fn = va_arg(arg_list, int*)) != NULL)
        {
            (*fn)();
        }
        va_end(arg_list);
    }
    
    void f1(){printf("one\n");}
    void f2(){printf("two\n");}
    void f3(){printf("three\n");}
    
    void main()
    {
        CallThese(f1,f2,f3,f1,f2,f3,f1);
    }

### Define case statements in a switch to reduce code redundancy
This is really useful on embedded systems where you may have multiple copies of a specific hardware block with the same initialization but different register names.

    void SpecificHardwareInit(uint8_t channel)
    {
    #define REG_INIT_CASE(x)    \
        case x:                 \
            REG_##x##_A = 0;    \
            REG_##x##_B = 0x55; \
            REG_##x##_C = 0x02; \
            break;

        switch(channel)
        {
            REG_INIT_CASE(0);
            REG_INIT_CASE(1);
            REG_INIT_CASE(2);
            REG_INIT_CASE(3);
        }
    #undef REG_INIT_CASE
    }

### Glue register names together to create optimized hardware init code
If you're working on a very space-limited device but still want clear hardware init code you can abuse the preprocessor and use it to glue together statements that compile to single statements.
    
    #define _BV(x)       (1<<(x))

    #define CLEAR_OP    &=~
    #define SET_OP      |=
    #define TOGGLE_OP   ^=

    #define _INPUT  CLEAR_OP
    #define _OUTPUT SET_OP

    #define HW_DIRECTION(n,op)    HW_DIRECTION_(n,op)
    #define HW_DIRECTION_(p,i,op) P##p##DIR op _BV(i)

    #define LED 1,3

    HW_DIRECTION(LED,GPIO_INPUT); // This line will be converted to P1DIR &=~(1<<(3));














