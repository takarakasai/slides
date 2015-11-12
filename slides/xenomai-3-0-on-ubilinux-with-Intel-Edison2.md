
クロスコンパイルもできますが、今回はedison上で行います

> script/bootstrap
> mkdir $build_root
> cd $build_root
> $xenomai_root/configure --with-core=cobalt --enable-smp --enable-pshared \
  --host=i686-linux CFLAGS="-m32 -O2" LDFLAGS="-m32"
> make
> make install

# test latency

root@ubilinux:/usr/xenomai/bin# ./latency -p 1000                                                                      
== Sampling period: 1000 us                                                                                            
== Test mode: periodic user-mode task                                                                                  
== All results in microseconds                                                                                         
warming up...                                                                                                          
RTT|  00:00:01  (periodic user-mode task, 1000 us period, priority 99)                                                 
RTH|----lat min|----lat avg|----lat max|-overrun|---msw|---lat best|--lat worst                                        
RTD|      1.443|      2.173|     10.737|       0|     0|      1.443|     10.737                                        
RTD|      1.513|      2.163|     10.787|       0|     0|      1.443|     10.787                                        
RTD|      1.502|      2.153|     10.587|       0|     0|      1.443|     10.787                                        
RTD|      1.452|      2.162|     11.038|       0|     0|      1.443|     11.038                                        
RTD|      1.502|      2.142|     10.907|       0|     0|      1.443|     11.038                                        
RTD|      1.482|      2.137|     10.527|       0|     0|      1.443|     11.038                                        
RTD|      1.542|      2.181|     10.577|       0|     0|      1.443|     11.038                                        
RTD|      1.422|      2.170|     10.456|       0|     0|      1.422|     11.038                                        
RTD|      1.522|      2.163|     10.717|       0|     0|      1.422|     11.038                                        
RTD|      1.532|      2.176|     10.757|       0|     0|      1.422|     11.038                                        
RTD|      1.542|      2.200|     10.947|       0|     0|      1.422|     11.038                                        
RTD|      1.462|      2.153|     11.007|       0|     0|      1.422|     11.038                                        
RTD|      1.512|      2.185|     10.826|       0|     0|      1.422|     11.038                                        
RTD|      0.850|      2.183|     10.696|       0|     0|      0.850|     11.038 

sudo apt-get install setserial
