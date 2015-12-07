---
layout: post
title: "Elementary Cellular Automata"
date: 2013-08-22 00:34
comments: true
categories: 
---

[Elementary Cellular Automaton](http://mathworld.wolfram.com/ElementaryCellularAutomaton.html) 를 1 라인으로 구현한 코드.

[Best one liner: konno.c](http://www.ioccc.org/2012/konno/konno.c)

konno.c

    int _;main(O,l,o)char**l,**o;{_++>>9||main(1&(o?(int)o:O)|O*2,l,
    putchar(_%32?atoi(1[l])>>(7&O<<!o>>!o+29)&32<_|_==16?35:32:10)%10);}

이것을 해독해보았다. 

    int pos=0;

    void p_b(int x)
    {
        int b[33];
        char *p;
        int i;
        unsigned int j;

        memset(b,0,33);
        for(i=32, j=(1L<<31); i>0; i--,j>>=1)
            *p++=(x&j) ==j ? '1':'0';
        return b;
    }

    main(evolution_seq,argv,prev_cell)
    int evolution_seq;
    char **argv;
    int prev_cell;
    {
        /* recurse from pos=1 to pos=512 */
        if (pos++ < 32*16) {
            int cell_color;

            /* Fill the 1st line with seed value */
            if (pos == 16)
                cell_color = '#';       /* The 1st line: the mid pos is '#' */
            else if (pos < 32)
                cell_color = ' ';       /* The 1st line: fill with ' ' except the mid pos */
            else if (pos % 32 == 0)
                cell_color = '\n';      /* Print newline for every 32nd pos */
            else {
                int rule;

                /* 
                * Get the hight-order 3 bits of evolution sequence
                * (clears MSB if the previous cell was new-line)
                * and shift rule bits by it.
                */
                if (prev_cell == '\n')
                    rule = atoi(argv[1]) >> (evolution_seq << 1 >> 30 & 0x7);  
                else
                    rule = atoi(argv[1]) >> (evolution_seq >> 29 & 0x7);

                if (rule & 0x1)
                    cell_color = '#'; /* black */
                else
                    cell_color = ' '; /* white */
            }        
            putchar(cell_color);

            if (prev_cell == '#') { 
                evolution_seq = evolution_seq * 2 + 1;
            } else 
                evolution_seq = evolution_seq * 2; 

            main(evolution_seq, argv, cell_color);
        }
    }

    ~% gcc k.c -o k
    ~% ./k 126
                   #               
                  ###              
                 ## ##             
                #######            
               ##     ##           
              ####   ####          
             ##  ## ##  ##         
            ###############        
           ##             ##       
          ####           ####      
         ##  ##         ##  ##     
        ########       ########    
       ##      ##     ##      ##   
      ####    ####   ####    ####  
     ##  ##  ##  ## ##  ##  ##  ## 
    ###############################

