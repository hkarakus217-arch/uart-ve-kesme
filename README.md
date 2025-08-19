# uart-ve-kesme

/* Includes ------------------------------------------------------------------*/

#include "main.h"

#include "string.h"

#include "i2c-lcd.h"

#include "stdio.h"


#define RX_BUFFER_SIZE 12

uint8_t rx_data[RX_BUFFER_SIZE];

char rx_string_buffer[RX_BUFFER_SIZE];

int buffer_index = 0;

I2C_HandleTypeDef hi2c1;

UART_HandleTypeDef huart1;


int main(void)
{

 
  MX_GPIO_Init();
 
  MX_I2C1_Init();
  
  MX_USART1_UART_Init();
  

  // LCD'yi baslat
  lcd_init();
  lcd_clear();
  lcd_send_string("Hazir!");
  
  // Ilk karakteri almak i�in UART alimini baslat
  HAL_UART_Receive_IT(&huart1, (uint8_t *)rx_data, RX_BUFFER_SIZE);

  
  while (1)
  {
    
	
		
  }
  
}


void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
  RCC_OscInitStruct.HSEState = RCC_HSE_ON;
  RCC_OscInitStruct.HSEPredivValue = RCC_HSE_PREDIV_DIV1;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
  RCC_OscInitStruct.PLL.PLLMUL = RCC_PLL_MUL9;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  /** Initializes the CPU, AHB and APB buses clocks
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV2;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;

  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_2) != HAL_OK)
  {
    Error_Handler();
  }
}

/**
  * @brief I2C1 Initialization Function
  * @param None
  * @retval None
  */
static void MX_I2C1_Init(void)
{

  /* USER CODE BEGIN I2C1_Init 0 */

  /* USER CODE END I2C1_Init 0 */

  /* USER CODE BEGIN I2C1_Init 1 */

  /* USER CODE END I2C1_Init 1 */
  hi2c1.Instance = I2C1;
  hi2c1.Init.ClockSpeed = 100000;
  hi2c1.Init.DutyCycle = I2C_DUTYCYCLE_2;
  hi2c1.Init.OwnAddress1 = 0;
  hi2c1.Init.AddressingMode = I2C_ADDRESSINGMODE_7BIT;
  hi2c1.Init.DualAddressMode = I2C_DUALADDRESS_DISABLE;
  hi2c1.Init.OwnAddress2 = 0;
  hi2c1.Init.GeneralCallMode = I2C_GENERALCALL_DISABLE;
  hi2c1.Init.NoStretchMode = I2C_NOSTRETCH_DISABLE;
  if (HAL_I2C_Init(&hi2c1) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN I2C1_Init 2 */

  /* USER CODE END I2C1_Init 2 */

}

/**
  * @brief USART1 Initialization Function
  * @param None
  * @retval None
  */
static void MX_USART1_UART_Init(void)
{


  huart1.Instance = USART1;
  huart1.Init.BaudRate = 115200;
  huart1.Init.WordLength = UART_WORDLENGTH_8B;
  huart1.Init.StopBits = UART_STOPBITS_1;
  huart1.Init.Parity = UART_PARITY_NONE;
  huart1.Init.Mode = UART_MODE_TX_RX;
  huart1.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart1.Init.OverSampling = UART_OVERSAMPLING_16;
  if (HAL_UART_Init(&huart1) != HAL_OK)
  {
    Error_Handler();
  }


}


void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	if (huart->Instance == huart1.Instance)
	{
	 // Her karakter aliminda LED'i yakip söndür
		HAL_UART_Transmit(&huart1, (uint8_t *)rx_data, sizeof(rx_data),RX_BUFFER_SIZE);
		if (rx_data[0] == '\r' || rx_data[0] == '\n')
		{
				rx_string_buffer[buffer_index] = '\0';
				HAL_UART_Transmit(&huart1, (uint8_t *)rx_data, sizeof(rx_data),RX_BUFFER_SIZE);
				lcd_clear();
				lcd_put_cur(0, 0);
				lcd_send_string(rx_string_buffer);
				
				buffer_index = 0;
		}
		else
		{
				if (buffer_index < RX_BUFFER_SIZE - 1)
				{
						rx_string_buffer[buffer_index++] = rx_data[0];
				}
		}
		HAL_UART_Receive_IT(&huart1, (uint8_t *)rx_data, RX_BUFFER_SIZE);
	}
}

