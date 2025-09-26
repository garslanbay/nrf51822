#include "nrf.h"
#include "nrf_drv_twi.h"
#include "ble_nus.h"
#include "app_timer.h"
#include "nrf_sdh.h"
#include "nrf_sdh_ble.h"
#include "nrf_delay.h"
#include <stdio.h>
#include <string.h>

#define TMP117_ADDR 0x48
#define READ_INTERVAL APP_TIMER_TICKS(1000)

APP_TIMER_DEF(m_temp_timer);
static const nrf_drv_twi_t m_twi = NRF_DRV_TWI_INSTANCE(0);
static ble_nus_t m_nus;
static uint16_t m_conn_handle = BLE_CONN_HANDLE_INVALID;

static void temperature_timeout_handler(void * p_context);
static void twi_init(void);
static int16_t tmp117_read_temperature(void);

void ble_evt_handler(ble_evt_t const * p_ble_evt, void * p_context) {
    switch (p_ble_evt->header.evt_id) {
        case BLE_GAP_EVT_CONNECTED:
            m_conn_handle = p_ble_evt->evt.gap_evt.conn_handle;
            break;
        case BLE_GAP_EVT_DISCONNECTED:
            m_conn_handle = BLE_CONN_HANDLE_INVALID;
            break;
        default:
            break;
    }
}

void ble_stack_init(void) {
    ret_code_t err_code;
    err_code = nrf_sdh_enable_request();
    APP_ERROR_CHECK(err_code);
    NRF_SDH_BLE_OBSERVER(m_ble_observer, 3, ble_evt_handler, NULL);
}

void twi_init(void) {
    ret_code_t err_code;
    const nrf_drv_twi_config_t twi_config = {
        .scl = 27,
        .sda = 26,
        .frequency = NRF_DRV_TWI_FREQ_100K,
        .interrupt_priority = APP_IRQ_PRIORITY_HIGH,
        .clear_bus_init = false
    };
    err_code = nrf_drv_twi_init(&m_twi, &twi_config, NULL, NULL);
    APP_ERROR_CHECK(err_code);
    nrf_drv_twi_enable(&m_twi);
}

int16_t tmp117_read_temperature(void) {
    uint8_t reg = 0x00;
    uint8_t temp_data[2];
    ret_code_t err_code;

    err_code = nrf_drv_twi_tx(&m_twi, TMP117_ADDR, &reg, 1, true);
    if (err_code != NRF_SUCCESS) return -10000;

    err_code = nrf_drv_twi_rx(&m_twi, TMP117_ADDR, temp_data, 2);
    if (err_code != NRF_SUCCESS) return -10000;

    return (int16_t)((temp_data[0] << 8) | temp_data[1]);
}

static void temperature_timeout_handler(void * p_context) {
    int16_t raw_temp = tmp117_read_temperature();
    if (raw_temp != -10000) {
        float temp_c = raw_temp * 0.0078125f;
        char temp_str[20];
        sprintf(temp_str, "Temp: %.2f C\r\n", temp_c);
        if (m_conn_handle != BLE_CONN_HANDLE_INVALID) {
            ret_code_t err_code = ble_nus_string_send(&m_nus, (uint8_t*)temp_str, strlen(temp_str));
            if (err_code != NRF_ERROR_INVALID_STATE) {
                APP_ERROR_CHECK(err_code);
            }
        }
    }
}

void timers_init(void) {
    ret_code_t err_code = app_timer_init();
    APP_ERROR_CHECK(err_code);
    err_code = app_timer_create(&m_temp_timer, APP_TIMER_MODE_REPEATED, temperature_timeout_handler);
    APP_ERROR_CHECK(err_code);
}

void application_timers_start(void) {
    ret_code_t err_code = app_timer_start(m_temp_timer, READ_INTERVAL, NULL);
    APP_ERROR_CHECK(err_code);
}

int main(void) {
    twi_init();
    ble_stack_init();
    timers_init();
    application_timers_start();

    while (true) {
        sd_app_evt_wait();
    }
}
