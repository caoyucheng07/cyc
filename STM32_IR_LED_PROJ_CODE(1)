#include "stm32f10x.h"
#include "OLED.h"

#define IR_RX_PIN      0
#define BTN_L1_PIN     1
#define BTN_S1_PIN     2
#define BTN_L2_PIN     3
#define BTN_S2_PIN     4

#define MAX_PULSES     200
#define NUM_SLOTS     2

volatile uint16_t ir_buffer[NUM_SLOTS][MAX_PULSES];
volatile uint16_t ir_len[NUM_SLOTS];
volatile uint8_t  ir_start_level[NUM_SLOTS];

void delay_us(uint16_t us) {
    TIM2->CNT = 0;
    while (TIM2->CNT < us);
}

void TIM2_Init(void) {
    RCC->APB1ENR |= RCC_APB1ENR_TIM2EN;
    TIM2->PSC = 72 - 1;     // 72 MHz / 72 = 1 MHz
    TIM2->ARR = 0xFFFF;
    TIM2->CR1 |= TIM_CR1_CEN;
}

#define IR_ON()   (TIM1->CR1 |= TIM_CR1_CEN)
#define IR_OFF()  (TIM1->CR1 &= ~TIM_CR1_CEN)

void IR_PWM_Init(void) {
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN | RCC_APB2ENR_TIM1EN;

    GPIOA->CRH &= ~(0xF << 0);
    GPIOA->CRH |=  (0xB << 0);

    TIM1->PSC = 0;
    TIM1->ARR  = 1799;
    TIM1->CCR1 = 900;
    TIM1->CCMR1 |= (6 << 4);
    TIM1->CCER  |= TIM_CCER_CC1E;
    TIM1->BDTR  |= TIM_BDTR_MOE;
    TIM1->CR1   |= TIM_CR1_ARPE;
    TIM1->EGR    = TIM_EGR_UG;
}


void GPIO_Init_All(void) {
     RCC->APB2ENR |= RCC_APB2ENR_IOPAEN
                 | RCC_APB2ENR_IOPBEN
                 | RCC_APB2ENR_AFIOEN;

    GPIOA->CRL &= ~(0xFFFFF);
    GPIOA->CRL |=  (0x88888);
    GPIOA->ODR |=  (0x1F);
}


void IR_Learn(uint8_t slot) {
    uint8_t last, now;

    ir_len[slot] = 0;
    ir_start_level[slot] =
        (GPIOA->IDR & (1 << IR_RX_PIN)) ? 1 : 0;

    last = ir_start_level[slot];
    TIM2->CNT = 0;

    while (ir_len[slot] < MAX_PULSES) {
        now = (GPIOA->IDR & (1 << IR_RX_PIN)) ? 1 : 0;

        if (now != last) {
            ir_buffer[slot][ir_len[slot]++] = TIM2->CNT;
            TIM2->CNT = 0;
            last = now;
        }

        if (TIM2->CNT > 50000 && ir_len[slot] > 10)
            break;
    }
		if (ir_len[slot] > 0)
			OLED_ShowString(slot+2,9,"O");
}


void IR_Send(uint8_t slot) {
    uint16_t i;
    uint8_t level = ir_start_level[slot];
		OLED_ShowString(slot+2,9,"Sending");

    for (i = 0; i < ir_len[slot]; i++) {
        if (level == 0) {
            IR_ON();
            delay_us(ir_buffer[slot][i]);
            IR_OFF();
        } else {
            IR_OFF();
            delay_us(ir_buffer[slot][i]);
        }
        level ^= 1;
    }
		OLED_ShowString(slot+2,9,"O      ");
    IR_OFF();
}


void IR_Send_Sony(uint8_t slot) {
    int r;
    for (r = 0; r < 3; r++) {
        IR_Send(slot);
        delay_us(30000);
    }
}


int main(void) {
    TIM2_Init();
    IR_PWM_Init();
    GPIO_Init_All();
		OLED_Init();
		delay_us(50000);
		OLED_Clear();
		OLED_ShowString(1,1,"Slots");
		OLED_ShowString(1,8,"Status");
		OLED_ShowString(2,1,"1");
		OLED_ShowString(3,1,"2");
		OLED_ShowString(2,9,"E");
		OLED_ShowString(3,9,"E");
    while (1) {

        if (!(GPIOA->IDR & (1 << BTN_L1_PIN))) {
            delay_us(20000);
            IR_Learn(0);
            while (!(GPIOA->IDR & (1 << BTN_L1_PIN)));
        }

        if (!(GPIOA->IDR & (1 << BTN_S1_PIN))) {
            delay_us(20000);
            IR_Send_Sony(0);
            while (!(GPIOA->IDR & (1 << BTN_S1_PIN)));
        }

        if (!(GPIOA->IDR & (1 << BTN_L2_PIN))) {
            delay_us(20000);
            IR_Learn(1);
            while (!(GPIOA->IDR & (1 << BTN_L2_PIN)));
        }

        if (!(GPIOA->IDR & (1 << BTN_S2_PIN))) {
            delay_us(20000);
            IR_Send_Sony(1);
            while (!(GPIOA->IDR & (1 << BTN_S2_PIN)));
        }
    }
}
