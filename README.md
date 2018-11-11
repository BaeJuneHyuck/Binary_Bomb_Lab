#Bomb_Lab  :: Solution by BaeJuneHyuck , PNU CSE

##<Phase_1>
00000000000012b4 <phase_1>:
    12b4:	48 83 ec 08          	sub    $0x8,%rsp
    12b8:	48 8d 35 d1 18 00 00 	lea    0x18d1(%rip),%rsi        # 2b90 <_IO_stdin_used+0x150>
    12bf:	e8 8c 05 00 00       	callq  1850 <strings_not_equal>
    12c4:	85 c0                	test   %eax,%eax
    12c6:	75 05                	jne    12cd <phase_1+0x19>
    12c8:	48 83 c4 08          	add    $0x8,%rsp
    12cc:	c3                   	retq   
    12cd:	e8 82 08 00 00       	callq  1b54 <explode_bomb>
    12d2:	eb f4                	jmp    12c8 <phase_1+0x14>

우선 Phase_1을 호출하는 메인 함수를 살펴보면 input = read_line()과 phase_1(input)을 확인 할 수 있다. 이후 다른 phase들에서도 동일하게 입력 받은 line (input)을 통해 각 phase를 호출하고 있음을 알 수 있다. 
이제 Phase_1로 들어가보면 스택의 공간을 할당 후 lea 0x18d1(%rip), %rsi을 통해 rip+0x18d1 즉 0x2b90의 값을 그대로 %rsi에 저장하고 있음을 확인할 수 있다. 이후     callq <strings_not_equal> 의 결과값(%eax)를 test하여 explode_bomb()으 로 점프할지, Phase_1이 정상적으로 종료될지가 결정된다. 앞의 정보들을 종합하여 %rsi를 통해 전달된 값이 input과 비교됨을 예상하였고, x/s 2b90을 입력해 그 주소에 있는 값이
"You can Russia from land here in Alaska." 라는 문자열임을 알게 되었다.
이를 입력하여 phase_1을 defuse하였다.

##<Phase_2>

00000000000012d4 <phase_2>:
    12d4:	55                   	push   %rbp
    12d5:	53                   	push   %rbx
    12d6:	48 83 ec 28          	sub    $0x28,%rsp
    12da:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
    12e1:	00 00 
    12e3:	48 89 44 24 18       	mov    %rax,0x18(%rsp)
    12e8:	31 c0                	xor    %eax,%eax
    12ea:	48 89 e6             	mov    %rsp,%rsi
    12ed:	e8 9e 08 00 00       	callq  1b90 <read_six_numbers>
    12f2:	83 3c 24 00          	cmpl   $0x0,(%rsp)
    12f6:	75 07                	jne    12ff <phase_2+0x2b>
    12f8:	83 7c 24 04 01       	cmpl   $0x1,0x4(%rsp)
    12fd:	74 05                	je     1304 <phase_2+0x30>
    12ff:	e8 50 08 00 00       	callq  1b54 <explode_bomb>
    1304:	48 89 e3             	mov    %rsp,%rbx
    1307:	48 8d 6b 10          	lea    0x10(%rbx),%rbp
    130b:	eb 09                	jmp    1316 <phase_2+0x42>
    130d:	48 83 c3 04          	add    $0x4,%rbx
    1311:	48 39 eb             	cmp    %rbp,%rbx
    1314:	74 11                	je     1327 <phase_2+0x53>
    1316:	8b 43 04             	mov    0x4(%rbx),%eax
    1319:	03 03                	add    (%rbx),%eax
    131b:	39 43 08             	cmp    %eax,0x8(%rbx)
    131e:	74 ed                	je     130d <phase_2+0x39>
    1320:	e8 2f 08 00 00       	callq  1b54 <explode_bomb>
    1325:	eb e6                	jmp    130d <phase_2+0x39>
    1327:	48 8b 44 24 18       	mov    0x18(%rsp),%rax
    132c:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
    1333:	00 00 
    1335:	75 07                	jne    133e <phase_2+0x6a>
    1337:	48 83 c4 28          	add    $0x28,%rsp
    133b:	5b                   	pop    %rbx
    133c:	5d                   	pop    %rbp
    133d:	c3                   	retq   
    133e:	e8 ad fb ff ff       	callq  ef0 <__stack_chk_fail@plt>

<read_six_numbers>를 통해 input된 line을 6개의 숫자로 저장하는 것을 알 수 있다. 이후 첫번째 비교인 cmpl $0x0, (%rsp) 를 통해 첫번째 입력은 0임을 알 수 있고, 다음 비교 cmpl  $0x1, 0x4(%rsp)를 통해 두번째 입력은 1임을 알 수 있다. 이후 jmp를 통해 함수의 중간으로 이동, 몇번의 명령을 수행 후 입력 받은 값과 이전 값들의 합을 비교하는 것을 확인 할 수 있고, 이를 통해 어떤 동작을 수행하는 loop문이 존재함을 알게 되었다. Loop 문 내부를 살펴보면add $0x4, %rbx / cmp %rbp, %rbx 이후 je를통해 Loop를 빠져나가는 것으로 보아 %rbx가 loop의 counter로써 작동하고 있음을 알 수 있다. Loop문 내부를 확인하면 다음의 명령들이 계속 반복된다.
mov    0x4(%rbx), %eax  	// %rbx+4
add    (%rbx), %eax     	// %rbx+4값 + %rbx
cmp    %eax,0x8(%rbx)	// 두개의 합과 입력한 수의 비교
이를 통해 x[i] = x[i-1]+x[i-2], 즉 피보나치 수열을 구현하는 알고리즘임을 알 수 있다
첫번째 두번째 조건인 x[0]=0, x[1]=1을 이용하면 정답 입력은0, 1, 1, 2, 3, 5이다.


##<Phase_3>


0000000000001343 <phase_3>:
    1343:	48 83 ec 28          	sub    $0x28,%rsp
    1347:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
    134e:	00 00 
    1350:	48 89 44 24 18       	mov    %rax,0x18(%rsp)
    1355:	31 c0                	xor    %eax,%eax
    1357:	48 8d 4c 24 0f       	lea    0xf(%rsp),%rcx
    135c:	48 8d 54 24 10       	lea    0x10(%rsp),%rdx
    1361:	4c 8d 44 24 14       	lea    0x14(%rsp),%r8
    1366:	48 8d 35 79 18 00 00 	lea    0x1879(%rip),%rsi        # 2be6 <_IO_stdin_used+0x1a6>
    136d:	e8 1e fc ff ff       	callq  f90 <__isoc99_sscanf@plt>
    1372:	83 f8 02             	cmp    $0x2,%eax
    1375:	7e 1f                	jle    1396 <phase_3+0x53>
    1377:	83 7c 24 10 07       	cmpl   $0x7,0x10(%rsp)
    137c:	0f 87 0c 01 00 00    	ja     148e <phase_3+0x14b>
    1382:	8b 44 24 10          	mov    0x10(%rsp),%eax
    1386:	48 8d 15 73 18 00 00 	lea    0x1873(%rip),%rdx        # 2c00 <_IO_stdin_used+0x1c0>
    138d:	48 63 04 82          	movslq (%rdx,%rax,4),%rax
    1391:	48 01 d0             	add    %rdx,%rax
    1394:	ff e0                	jmpq   *%rax
    1396:	e8 b9 07 00 00       	callq  1b54 <explode_bomb>
    139b:	eb da                	jmp    1377 <phase_3+0x34>
    139d:	b8 79 00 00 00       	mov    $0x79,%eax
    13a2:	81 7c 24 14 98 03 00 	cmpl   $0x398,0x14(%rsp)
    13a9:	00 
    13aa:	0f 84 e8 00 00 00    	je     1498 <phase_3+0x155>
    13b0:	e8 9f 07 00 00       	callq  1b54 <explode_bomb>
    13b5:	b8 79 00 00 00       	mov    $0x79,%eax
    13ba:	e9 d9 00 00 00       	jmpq   1498 <phase_3+0x155>
    13bf:	b8 78 00 00 00       	mov    $0x78,%eax
    13c4:	81 7c 24 14 10 03 00 	cmpl   $0x310,0x14(%rsp)
    13cb:	00 
    13cc:	0f 84 c6 00 00 00    	je     1498 <phase_3+0x155>
    13d2:	e8 7d 07 00 00       	callq  1b54 <explode_bomb>
    13d7:	b8 78 00 00 00       	mov    $0x78,%eax
    13dc:	e9 b7 00 00 00       	jmpq   1498 <phase_3+0x155>
    13e1:	b8 6d 00 00 00       	mov    $0x6d,%eax
    13e6:	81 7c 24 14 47 01 00 	cmpl   $0x147,0x14(%rsp)
    13ed:	00 
    13ee:	0f 84 a4 00 00 00    	je     1498 <phase_3+0x155>
    13f4:	e8 5b 07 00 00       	callq  1b54 <explode_bomb>
    13f9:	b8 6d 00 00 00       	mov    $0x6d,%eax
    13fe:	e9 95 00 00 00       	jmpq   1498 <phase_3+0x155>
    1403:	b8 69 00 00 00       	mov    $0x69,%eax
    1408:	81 7c 24 14 94 02 00 	cmpl   $0x294,0x14(%rsp)
    140f:	00 
    1410:	0f 84 82 00 00 00    	je     1498 <phase_3+0x155>
    1416:	e8 39 07 00 00       	callq  1b54 <explode_bomb>
    141b:	b8 69 00 00 00       	mov    $0x69,%eax
    1420:	eb 76                	jmp    1498 <phase_3+0x155>
    1422:	b8 77 00 00 00       	mov    $0x77,%eax
    1427:	81 7c 24 14 67 01 00 	cmpl   $0x167,0x14(%rsp)
    142e:	00 
    142f:	74 67                	je     1498 <phase_3+0x155>
    1431:	e8 1e 07 00 00       	callq  1b54 <explode_bomb>
    1436:	b8 77 00 00 00       	mov    $0x77,%eax
    143b:	eb 5b                	jmp    1498 <phase_3+0x155>
    143d:	b8 61 00 00 00       	mov    $0x61,%eax
    1442:	81 7c 24 14 48 01 00 	cmpl   $0x148,0x14(%rsp)
    1449:	00 
    144a:	74 4c                	je     1498 <phase_3+0x155>
    144c:	e8 03 07 00 00       	callq  1b54 <explode_bomb>
    1451:	b8 61 00 00 00       	mov    $0x61,%eax
    1456:	eb 40                	jmp    1498 <phase_3+0x155>
    1458:	b8 65 00 00 00       	mov    $0x65,%eax
    145d:	81 7c 24 14 b0 03 00 	cmpl   $0x3b0,0x14(%rsp)
    1464:	00 
    1465:	74 31                	je     1498 <phase_3+0x155>
    1467:	e8 e8 06 00 00       	callq  1b54 <explode_bomb>
    146c:	b8 65 00 00 00       	mov    $0x65,%eax
    1471:	eb 25                	jmp    1498 <phase_3+0x155>
    1473:	b8 6a 00 00 00       	mov    $0x6a,%eax
    1478:	81 7c 24 14 5e 03 00 	cmpl   $0x35e,0x14(%rsp)
    147f:	00 
    1480:	74 16                	je     1498 <phase_3+0x155>
    1482:	e8 cd 06 00 00       	callq  1b54 <explode_bomb>
    1487:	b8 6a 00 00 00       	mov    $0x6a,%eax
    148c:	eb 0a                	jmp    1498 <phase_3+0x155>
    148e:	e8 c1 06 00 00       	callq  1b54 <explode_bomb>
    1493:	b8 7a 00 00 00       	mov    $0x7a,%eax
    1498:	38 44 24 0f          	cmp    %al,0xf(%rsp)
    149c:	74 05                	je     14a3 <phase_3+0x160>
    149e:	e8 b1 06 00 00       	callq  1b54 <explode_bomb>
    14a3:	48 8b 44 24 18       	mov    0x18(%rsp),%rax
    14a8:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
    14af:	00 00 
    14b1:	75 05                	jne    14b8 <phase_3+0x175>
    14b3:	48 83 c4 28          	add    $0x28,%rsp
    14b7:	c3                   	retq   
    14b8:	e8 33 fa ff ff       	callq  ef0 <__stack_chk_fail@plt>

 scanf 실행 직전에 lea 0x1879(%rip), %rsi를 실행하여 format으로 rip(136d) + 1879 = 2be6 을 전달하고 있으며 x/s 0x2be6을 통해 이 값이 "%b %c %d”임을 확인하였음. 첫번째 비교인 
cmp $0x2, %eax 를 통해 scanf의 리턴인 %eax가 2보다 큰 점을 알 수 있는데, scanf의 리턴은 입력의 개수이므로 “%b %c %d”를 입력 해야함을 재확인할 수 있다. 두번째 비교인 cmpl $0x7, 0x10(%rsp)을 통해 입력이 7보다 클 경우 <explode_bomb>으로 점프하는 것을 확인하였다. 이후 movslq (%rdx, %rax,4), %rax / jmpq *%rax를 통해 스위치문의 내부로 간접 점프함을 확인할 수 있다. 첫번째 입력으로 1을 넣을 경우 mov $0x78, %eax 로 시작하는 케이스문으로 들어가는데, 0x78은 char로 x와 같고 이후cmpl $0x310, 0x14(%rsp)의 비교를 통해 다음 입력되는 정수가 0x310, 즉 784와 비교됨을 알 수 있다. 따라서 1 x 784를 입력, Phase_3를 defuse 하였다.

##<Phase_4>

00000000000014bd <func4>:
    14bd:	b8 00 00 00 00       	mov    $0x0,%eax
    14c2:	85 ff                	test   %edi,%edi
    14c4:	7e 07                	jle    14cd <func4+0x10>
    14c6:	89 f0                	mov    %esi,%eax
    14c8:	83 ff 01             	cmp    $0x1,%edi
    14cb:	75 02                	jne    14cf <func4+0x12>
    14cd:	f3 c3                	repz retq 
    14cf:	41 54                	push   %r12
    14d1:	55                   	push   %rbp
    14d2:	53                   	push   %rbx
    14d3:	41 89 f4             	mov    %esi,%r12d
    14d6:	89 fb                	mov    %edi,%ebx
    14d8:	8d 7f ff             	lea    -0x1(%rdi),%edi
    14db:	e8 dd ff ff ff       	callq  14bd <func4>
    14e0:	42 8d 2c 20          	lea    (%rax,%r12,1),%ebp
    14e4:	8d 7b fe             	lea    -0x2(%rbx),%edi
    14e7:	44 89 e6             	mov    %r12d,%esi
    14ea:	e8 ce ff ff ff       	callq  14bd <func4>
    14ef:	01 e8                	add    %ebp,%eax
    14f1:	5b                   	pop    %rbx
    14f2:	5d                   	pop    %rbp
    14f3:	41 5c                	pop    %r12
    14f5:	c3                   	retq   

00000000000014f6 <phase_4>:
    14f6:	48 83 ec 18          	sub    $0x18,%rsp
    14fa:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
    1501:	00 00 
    1503:	48 89 44 24 08       	mov    %rax,0x8(%rsp)
    1508:	31 c0                	xor    %eax,%eax
    150a:	48 89 e1             	mov    %rsp,%rcx
    150d:	48 8d 54 24 04       	lea    0x4(%rsp),%rdx
    1512:	48 8d 35 94 19 00 00 	lea    0x1994(%rip),%rsi        # 2ead <array.3416+0x28d>
    1519:	e8 72 fa ff ff       	callq  f90 <__isoc99_sscanf@plt>
    151e:	83 f8 02             	cmp    $0x2,%eax
    1521:	75 0b                	jne    152e <phase_4+0x38>
    1523:	8b 04 24             	mov    (%rsp),%eax
    1526:	83 e8 02             	sub    $0x2,%eax
    1529:	83 f8 02             	cmp    $0x2,%eax
    152c:	76 05                	jbe    1533 <phase_4+0x3d>
    152e:	e8 21 06 00 00       	callq  1b54 <explode_bomb>
    1533:	8b 34 24             	mov    (%rsp),%esi
    1536:	bf 08 00 00 00       	mov    $0x8,%edi
    153b:	e8 7d ff ff ff       	callq  14bd <func4>
    1540:	39 44 24 04          	cmp    %eax,0x4(%rsp)
    1544:	74 05                	je     154b <phase_4+0x55>
    1546:	e8 09 06 00 00       	callq  1b54 <explode_bomb>
    154b:	48 8b 44 24 08       	mov    0x8(%rsp),%rax
    1550:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
    1557:	00 00 
    1559:	75 05                	jne    1560 <phase_4+0x6a>
    155b:	48 83 c4 18          	add    $0x18,%rsp
    155f:	c3                   	retq   
    1560:	e8 8b f9 ff ff       	callq  ef0 <__stack_chk_fail@plt>


 Phase_4의 도입부의 lea 0x1994(%rip), %rsi를 확인, x/s 0x2EAD를 통해 입력이 “%d %d”임을 알 수 있다. 첫번째 비교를 수행하는 3개의 명령어 mov (%rsp), %eax / sub  $0x2, %eax / cmp   $0x2, %eax를 통해 하나의 입력은 4이하임을 알 수 있다. 이후 Phase_4는 func4() 라는 또다른 함수를 인자8로 호출한다.
 func4()를 살펴보면 재귀적으로 자기 자신을 호출하는데. 인자가 0이면 그대로 0을 리턴, 인자가 1일경우 %rsi의 값이 리턴. 그 외의 경우는 아래와 같이
lea    -0x1(%rdi), %edi / callq  14bd <func4> 를 통해 func4(x-1)을
lea    -0x2(%rdi), %edi / callq  14bd <func4> 를 통해 func4(x-2)를 호출, 두 결과를 %rsi와 더해주고 있다. 이를 통해 x>1일때, func4(x) = funx4(x-1)+func4(x-2)+%rsi를 알 수 있다.
다시Phase_4로 돌아오면 cmp %eax, 0x4(%rsp)를 통해 func4의 리턴값과 두번째 입력을 비교함을 알 수 있다. 조건에 맞는 두번째 입력으로 2를 입력 후, Func4(8)의 리턴값을 계산하면 108을 얻을 수 있으며. 이에 108 2를 입력하여 Phase_4를 defuse 하였다.


##<Phase_5>

0000000000001565 <phase_5>:
    1565:	48 83 ec 18          	sub    $0x18,%rsp
    1569:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
    1570:	00 00 
    1572:	48 89 44 24 08       	mov    %rax,0x8(%rsp)
    1577:	31 c0                	xor    %eax,%eax
    1579:	48 8d 4c 24 04       	lea    0x4(%rsp),%rcx
    157e:	48 89 e2             	mov    %rsp,%rdx
    1581:	48 8d 35 25 19 00 00 	lea    0x1925(%rip),%rsi        # 2ead <array.3416+0x28d>
    1588:	e8 03 fa ff ff       	callq  f90 <__isoc99_sscanf@plt>
    158d:	83 f8 01             	cmp    $0x1,%eax
    1590:	7e 5a                	jle    15ec <phase_5+0x87>
    1592:	8b 04 24             	mov    (%rsp),%eax
    1595:	83 e0 0f             	and    $0xf,%eax
    1598:	89 04 24             	mov    %eax,(%rsp)
    159b:	83 f8 0f             	cmp    $0xf,%eax
    159e:	74 32                	je     15d2 <phase_5+0x6d>
    15a0:	b9 00 00 00 00       	mov    $0x0,%ecx
    15a5:	ba 00 00 00 00       	mov    $0x0,%edx
    15aa:	48 8d 35 6f 16 00 00 	lea    0x166f(%rip),%rsi        # 2c20 <array.3416>
    15b1:	83 c2 01             	add    $0x1,%edx
    15b4:	48 98                	cltq   
    15b6:	8b 04 86             	mov    (%rsi,%rax,4),%eax
    15b9:	01 c1                	add    %eax,%ecx
    15bb:	83 f8 0f             	cmp    $0xf,%eax
    15be:	75 f1                	jne    15b1 <phase_5+0x4c>
    15c0:	c7 04 24 0f 00 00 00 	movl   $0xf,(%rsp)
    15c7:	83 fa 0f             	cmp    $0xf,%edx
    15ca:	75 06                	jne    15d2 <phase_5+0x6d>
    15cc:	39 4c 24 04          	cmp    %ecx,0x4(%rsp)
    15d0:	74 05                	je     15d7 <phase_5+0x72>
    15d2:	e8 7d 05 00 00       	callq  1b54 <explode_bomb>
    15d7:	48 8b 44 24 08       	mov    0x8(%rsp),%rax
    15dc:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
    15e3:	00 00 
    15e5:	75 0c                	jne    15f3 <phase_5+0x8e>
    15e7:	48 83 c4 18          	add    $0x18,%rsp
    15eb:	c3                   	retq   
    15ec:	e8 63 05 00 00       	callq  1b54 <explode_bomb>
    15f1:	eb 9f                	jmp    1592 <phase_5+0x2d>
    15f3:	e8 f8 f8 ff ff       	callq  ef0 <__stack_chk_fail@plt>




 lea 0x1925(%rip), %rsi 에서x/s 0x2ead 을 통해 입력의 형식이 “%d %d”임을 확인 하였다. mov (%rsp), %eax / and $0xf, %eax / mov %eax, (%rsp) / cmp $0xf, %eax 의 일련의 명령을 통해 첫번째 입력은 어떤 값을 주던지 간에 최하위 4비트만 저장되며, 그 값이 1111은 아님을 알 수 있다. 
루프 직전의 lea 0x166f(%rip), %rsi에서 계산되는 주소를 x/16dw 로 출력하면 아래와 같다.

0x555555556c20 <array.3416>    :  10  2  14  7 
0x555555556c30 <array.3416+16>:  8  12  15  11
0x555555556c40 <array.3416+32>:  0  4  1  13  
0x555555556c50 <array.3416+48>:  3  9  6  5   

%rsi에 크기가 16이며 그 값들이 0~15사이의 유일한 정수인 배열이 들어가는 것을 알 수 있다. 이제 루프 내부로 들어가보면 아래의 명령들을 확인할 수 있는데,
mov   (%rsi,%rax,4), %eax 	// %rax를 offset으로 배열의 값을 얻어오고
add   %eax, %ecx			// %ecx에 그 값을 계속 더해주고 있으며
cmp   $0xf, %eax			// %eax가 0xf일경우 루프를 빠져나온다
루프를 빠져 나온 뒤 cmp $0xf, %edx에 의해 정확히 15번 루프문을 반복해야 하는 추가 조건을 확인하였다. 이를 종합하면 15번째 반복에서 15의 값을 갖는 offset(=6)에 도착해야 하는 것을 알 수 있다.
5를 입력하면 15번의 반복 후 루프문을 빠져나오며 그 반복동안 합해진 %ecx의값은 115이다. 배열을 참조하는 과정은 아래와 같다.
arr[5] = 12 -> arr[12]= 3 -> arr[3] = 7 -> arr[7] = 11 -> arr[11] = 13 -> arr[13] = 9 -> arr[9] = 4 -> arr[4] = 8 -> arr[8] = 0 -> arr[0] = 10 -> arr[10] = 1 -> arr[1]=2 -> arr[2] =14 -> arr[14] = 6 -> arr[6] = 15 
따라서 5 115를 입력하면 <Phase_5>를 defuse 할 수 있다.


##<phase_6>



00000000000015f8 <phase_6>:
    15f8:	41 55                	push   %r13
    15fa:	41 54                	push   %r12
    15fc:	55                   	push   %rbp
    15fd:	53                   	push   %rbx
    15fe:	48 83 ec 68          	sub    $0x68,%rsp
    1602:	64 48 8b 04 25 28 00 	mov    %fs:0x28,%rax
    1609:	00 00 
    160b:	48 89 44 24 58       	mov    %rax,0x58(%rsp)
    1610:	31 c0                	xor    %eax,%eax
    1612:	49 89 e4             	mov    %rsp,%r12
    1615:	4c 89 e6             	mov    %r12,%rsi
    1618:	e8 73 05 00 00       	callq  1b90 <read_six_numbers>
    161d:	41 bd 00 00 00 00    	mov    $0x0,%r13d
    1623:	eb 25                	jmp    164a <phase_6+0x52>
    1625:	e8 2a 05 00 00       	callq  1b54 <explode_bomb>
    162a:	eb 2d                	jmp    1659 <phase_6+0x61>
    162c:	83 c3 01             	add    $0x1,%ebx
    162f:	83 fb 05             	cmp    $0x5,%ebx
    1632:	7f 12                	jg     1646 <phase_6+0x4e>
    1634:	48 63 c3             	movslq %ebx,%rax
    1637:	8b 04 84             	mov    (%rsp,%rax,4),%eax
    163a:	39 45 00             	cmp    %eax,0x0(%rbp)
    163d:	75 ed                	jne    162c <phase_6+0x34>
    163f:	e8 10 05 00 00       	callq  1b54 <explode_bomb>
    1644:	eb e6                	jmp    162c <phase_6+0x34>
    1646:	49 83 c4 04          	add    $0x4,%r12
    164a:	4c 89 e5             	mov    %r12,%rbp
    164d:	41 8b 04 24          	mov    (%r12),%eax
    1651:	83 e8 01             	sub    $0x1,%eax
    1654:	83 f8 05             	cmp    $0x5,%eax
    1657:	77 cc                	ja     1625 <phase_6+0x2d>
    1659:	41 83 c5 01          	add    $0x1,%r13d
    165d:	41 83 fd 06          	cmp    $0x6,%r13d
    1661:	74 35                	je     1698 <phase_6+0xa0>
    1663:	44 89 eb             	mov    %r13d,%ebx
    1666:	eb cc                	jmp    1634 <phase_6+0x3c>
    1668:	48 8b 52 08          	mov    0x8(%rdx),%rdx
    166c:	83 c0 01             	add    $0x1,%eax
    166f:	39 c8                	cmp    %ecx,%eax
    1671:	75 f5                	jne    1668 <phase_6+0x70>
    1673:	48 89 54 f4 20       	mov    %rdx,0x20(%rsp,%rsi,8)
    1678:	48 83 c6 01          	add    $0x1,%rsi
    167c:	48 83 fe 06          	cmp    $0x6,%rsi
    1680:	74 1d                	je     169f <phase_6+0xa7>
    1682:	8b 0c b4             	mov    (%rsp,%rsi,4),%ecx
    1685:	b8 01 00 00 00       	mov    $0x1,%eax
    168a:	48 8d 15 9f 2b 20 00 	lea    0x202b9f(%rip),%rdx        # 204230 <node1>
    1691:	83 f9 01             	cmp    $0x1,%ecx
    1694:	7f d2                	jg     1668 <phase_6+0x70>
    1696:	eb db                	jmp    1673 <phase_6+0x7b>
    1698:	be 00 00 00 00       	mov    $0x0,%esi
    169d:	eb e3                	jmp    1682 <phase_6+0x8a>
    169f:	48 8b 5c 24 20       	mov    0x20(%rsp),%rbx
    16a4:	48 8b 44 24 28       	mov    0x28(%rsp),%rax
    16a9:	48 89 43 08          	mov    %rax,0x8(%rbx)
    16ad:	48 8b 54 24 30       	mov    0x30(%rsp),%rdx
    16b2:	48 89 50 08          	mov    %rdx,0x8(%rax)
    16b6:	48 8b 44 24 38       	mov    0x38(%rsp),%rax
    16bb:	48 89 42 08          	mov    %rax,0x8(%rdx)
    16bf:	48 8b 54 24 40       	mov    0x40(%rsp),%rdx
    16c4:	48 89 50 08          	mov    %rdx,0x8(%rax)
    16c8:	48 8b 44 24 48       	mov    0x48(%rsp),%rax
    16cd:	48 89 42 08          	mov    %rax,0x8(%rdx)
    16d1:	48 c7 40 08 00 00 00 	movq   $0x0,0x8(%rax)
    16d8:	00 
    16d9:	bd 05 00 00 00       	mov    $0x5,%ebp
    16de:	eb 09                	jmp    16e9 <phase_6+0xf1>
    16e0:	48 8b 5b 08          	mov    0x8(%rbx),%rbx
    16e4:	83 ed 01             	sub    $0x1,%ebp
    16e7:	74 11                	je     16fa <phase_6+0x102>
    16e9:	48 8b 43 08          	mov    0x8(%rbx),%rax
    16ed:	8b 00                	mov    (%rax),%eax
    16ef:	39 03                	cmp    %eax,(%rbx)
    16f1:	7e ed                	jle    16e0 <phase_6+0xe8>
    16f3:	e8 5c 04 00 00       	callq  1b54 <explode_bomb>
    16f8:	eb e6                	jmp    16e0 <phase_6+0xe8>
    16fa:	48 8b 44 24 58       	mov    0x58(%rsp),%rax
    16ff:	64 48 33 04 25 28 00 	xor    %fs:0x28,%rax
    1706:	00 00 
    1708:	75 0b                	jne    1715 <phase_6+0x11d>
    170a:	48 83 c4 68          	add    $0x68,%rsp
    170e:	5b                   	pop    %rbx
    170f:	5d                   	pop    %rbp
    1710:	41 5c                	pop    %r12
    1712:	41 5d                	pop    %r13
    1714:	c3                   	retq   
    1715:	e8 d6 f7 ff ff       	callq  ef0 <__stack_chk_fail@plt>


<read_six_numbers>이후 여러 번의 비교를 통해 6개의 입력된 정수가 1이상, 6이하임을 알 수 있고lea  0x202b9f(%rip), %rdx에서 계산되는 주소를 참조하여 이하의 노드들의 존재를 확인하였다.
	node		value		#node	 	pointer to next node
x/3x 0x555555758230	3ae		1		0x555555758240
x/3x 0x555555758240	2e9		2		0x555555758250
x/3x 0x555555758250	33b		3		0x555555758260
x/3x 0x555555758260	129		4		0x555555758270
x/3x 0x555555758270	fa		5		0x555555758110
x/3x 0x555555758110	127		6		0000000000	
이후 mov 0x20(%rsp), %rbx 와 같은 여러 번의 주소 참조 mov를 통해 입력 받은 값을 노드 방문 순서로 사용, 각 노드들을 순회하며 현재 노드의 value가 다음 노드의 value보다 크거나 같을 경우 <explode_bomb>으로 점프하는 것을 확인 할 수 있다. 따라서 노드의 값이 작은 순서대로 입력을 주면 폭탄이 터지는 것을 피할 수 있으므로, 5 6 4 2 3 1을 입력하여 Phase_6를 defuse 하였다.

##<Secret_Phase>


000000000000171a <fun7>:
    171a:	48 85 ff             	test   %rdi,%rdi
    171d:	74 34                	je     1753 <fun7+0x39>
    171f:	48 83 ec 08          	sub    $0x8,%rsp
    1723:	8b 17                	mov    (%rdi),%edx
    1725:	39 f2                	cmp    %esi,%edx
    1727:	7f 0e                	jg     1737 <fun7+0x1d>
    1729:	b8 00 00 00 00       	mov    $0x0,%eax
    172e:	39 f2                	cmp    %esi,%edx
    1730:	75 12                	jne    1744 <fun7+0x2a>
    1732:	48 83 c4 08          	add    $0x8,%rsp
    1736:	c3                   	retq   
    1737:	48 8b 7f 08          	mov    0x8(%rdi),%rdi
    173b:	e8 da ff ff ff       	callq  171a <fun7>
    1740:	01 c0                	add    %eax,%eax
    1742:	eb ee                	jmp    1732 <fun7+0x18>
    1744:	48 8b 7f 10          	mov    0x10(%rdi),%rdi
    1748:	e8 cd ff ff ff       	callq  171a <fun7>
    174d:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
    1751:	eb df                	jmp    1732 <fun7+0x18>
    1753:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
    1758:	c3                   	retq   

0000000000001759 <secret_phase>:
    1759:	53                   	push   %rbx
    175a:	e8 72 04 00 00       	callq  1bd1 <read_line>
    175f:	ba 0a 00 00 00       	mov    $0xa,%edx
    1764:	be 00 00 00 00       	mov    $0x0,%esi
    1769:	48 89 c7             	mov    %rax,%rdi
    176c:	e8 ff f7 ff ff       	callq  f70 <strtol@plt>
    1771:	48 89 c3             	mov    %rax,%rbx
    1774:	8d 40 ff             	lea    -0x1(%rax),%eax
    1777:	3d e8 03 00 00       	cmp    $0x3e8,%eax
    177c:	77 2b                	ja     17a9 <secret_phase+0x50>
    177e:	89 de                	mov    %ebx,%esi
    1780:	48 8d 3d c9 29 20 00 	lea    0x2029c9(%rip),%rdi        # 204150 <n1>
    1787:	e8 8e ff ff ff       	callq  171a <fun7>
    178c:	83 f8 01             	cmp    $0x1,%eax
    178f:	74 05                	je     1796 <secret_phase+0x3d>
    1791:	e8 be 03 00 00       	callq  1b54 <explode_bomb>
    1796:	48 8d 3d 23 14 00 00 	lea    0x1423(%rip),%rdi        # 2bc0 <_IO_stdin_used+0x180>
    179d:	e8 2e f7 ff ff       	callq  ed0 <puts@plt>
    17a2:	e8 6e 05 00 00       	callq  1d15 <phase_defused>
    17a7:	5b                   	pop    %rbx
    17a8:	c3                   	retq   
    17a9:	e8 a6 03 00 00       	callq  1b54 <explode_bomb>
    17ae:	eb ce                	jmp    177e <secret_phase+0x25>



각각의 phase를 defuse할때 호출되는 phase_defuse()를 살펴보면 phase_4의 정답은 두개인데 “%d %d %s”를 입력하면 추가적인 secret_phase()가 호출됨을 알 수 있다. %s가 비교되는 0x 555555556f00의 값을 x/s 명령어를 통해 입력, 문자열 “DrEvil”을 확인하였다. Phase_4의 입력 시 추가로 “DrEvil”을 입력하면 Secret_Phase로 진입할 수 있다.
Secret_Phase를 확인하면 callq 171a <fun7>  /  cmp $0x1, %eax  의 핵심코드를 가지고 있다. %eax (func7의 리턴값)이 1이기만 하면 폭탄을 defuse 가능한 것이다. 
func7은 재귀함수로써 %esi(주어진 입력)과 현재 노드의 주소를 입력으로 받고, 노드(이하 pNode)가 NULL일경우 -1, 노드의 값이 입력보다 클 경우 func7(esi, pNode->left)*2, 같을 경우0, 작을 경우 func7(esi, pNode->right) * 2 + 1을 각각 리턴한다. 

입력으로 사용되는 노드는 lea 0x2029c9(%rip),%rdi를 통해 참조되는 것을 확인할 수 있고, x/120wd 0x555555758150 을 통해 출력된 내용의 일부는 아래와 같다.

0x555555758150 <n1>:    0x00000024      0x00000000      0x55758170      0x00005555
0x555555758160 <n1+16>: 0x55758190      0x00005555      0x00000000      0x00000000
0x555555758170 <n21>:   0x00000008      0x00000000      0x557581f0      0x00005555
0x555555758180 <n21+16>:0x557581b0      0x00005555      0x00000000      0x00000000 

즉, 노드들은 value, left*, right*로 구성된다. 모든 노드들을 따라가 확인하면 각각의 노드가 tree형태로 연결된 것을 확인 할 수 있으며, 이를 표현하면 아래와 같다.
						n1(24)
		         n21(8) 		                 n22(32)
	      n31(6)           n32(16)		    n33(2d)          n44(23)
 	n41(1)   n42(7)   n43(14)  n44(23)	n45(28)  n46(2f)   (null)   (null)

fun7의 알고리즘을 적용하여 1을 리턴하는 값을 입력해야 하는데, 45를 입력하면 아래와 같은 재귀호출들이 이루어진다.
 1) n1의 값 24(10진수 36)는 45보다 작으므로return fun7(45,n1->right) * 2 + 1
 2) n22의 값32(10진수 50)는 45보다 크기에 return fun7(45,n22->left) * 2
 3) 2d(10진수 45)와 같으니 0(재귀호출 종료).  즉( 0*2+1) = 1을 리턴하여 조건에 의해 phase_secret을 빠져나올 수 있다.
따라서 45를 입력하여 secret_phase를 defuse하였다.




