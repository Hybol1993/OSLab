os1.c代码说明
=============
<center>毛海宇    2015310607</center>

------------------------------------------------------
'
task0()
{
  while(current < 10) {
    write(1, "0", 1);
  }
  write(1,"task0 exit\n", 11);
  halt(0);//挂起系统
}
'

task1()
{
  while(current < 10) {
    write(1, "1", 1);
  }
  write(1,"task1 exit\n", 11);
  halt(0);
}

swtch(int *old, int new) // switch stacks
{
  asm(LEA, 0); // a = sp
  asm(LBL, 8); // b = old
  asm(SX, 0);  // *b = a
  asm(LL, 16); // a = new
  asm(SSP);    // sp = a
}

trap()
{
  if (++current & 1)
    swtch(&task0_sp, task1_sp);// 任务切换，通过current的奇偶，切换stack
  else
    swtch(&task1_sp, task0_sp);
}

alltraps()
{
  //压栈，保护现场
  asm(PSHA);  // sp -= 8; *sp = a;
  asm(PSHB);  // sp -= 8; *sp = b;
  asm(PSHC);  // sp -= 8; *sp = c;

  //陷阱门
  trap();
  //从栈中恢复所有寄存器的值，恢复现场，中断返回。到中断打断的地方继续执行
  asm(POPC);  // c = *sp; sp += 8
  asm(POPB);  // b = *sp; sp += 8
  asm(POPA);  // a = *sp; sp += 8;
  asm(RTI);  // return from interrupt, POP fault code, pc, sp,  if fault code== USER, then switch to user mode; if has pending interrupt, process the interrupt

}

trapret() // 完成对返回前的寄存器和栈的恢复准备工作，最后通过RTI中断返回指令回到中断打断的地方继续执行。
{
  asm(POPC);
  asm(POPB);
  asm(POPA);
  asm(RTI);  //中断返回
}

main()
{
  current = 0;
  stmr(5000); //timer interrupt 计时器timer大于设置的超时时间阈值timeout，并且中断使能iena=1, 此时将falut code 置为FIMER, iena = 0, 并跳转至中断处理，即pc += ivec;
  // PSHI (5000) : sp -= 8; *sp=imm (立即数);
  // JSR:  *sp = pc; sp -= 8; pc += imm;
  // ENT:  sp += imm;
  ivec(alltraps);// entry address of interrupt
  //如果
  task1_sp = &task1_stack; //array point;
  task1_sp += 50;  // task1_stack[50];


  task1_sp -= 2; *task1_sp = &task1;// task1_stack[48] = &task1;
  task1_sp -= 2; *task1_sp = 0; // fault code;
  task1_sp -= 2; *task1_sp = 0; // a
  task1_sp -= 2; *task1_sp = 0; // b
  task1_sp -= 2; *task1_sp = 0; // c
  task1_sp -= 2; *task1_sp = &trapret; //task1_stack[38] = &trapret;

  asm(STI); //start interrupt 中断使能。

  task0(); //return halt(0)  printf( task0 exit）；
}


