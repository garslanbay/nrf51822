# nrf51822
sdk_config.h
İçeriği Nordic SDK içindeki örneklerin sdk_config.h dosyasından alabilir, aşağıdaki modülleri aktif edebilirsiniz:

#define NRF_DRV_TWI_ENABLED 1

#define APP_TIMER_ENABLED 1

#define BLE_NUS_ENABLED 1

#define NRF_SDH_BLE_ENABLED 1

#define NRF_SDH_ENABLED 1

Diğer dosyalar
system_nrf51.c/h — Nordic SDK içinde bulunur.

linker_script.sct — Nordic SDK örneklerinden alınabilir.

ble_stack.c/h — BLE başlatma, servis yapılandırma Nordic SDK örneklerinden alınabilir.

twi_handler.c/h — nrf_drv_twi modülünü kullanarak I2C fonksiyonlarını soyutlayan dosyalar.

Projeyi Oluşturma
Keil MDK açın, yeni proje oluşturun,

NRF51822 cihazını seçin,

Nordic SDK’dan yukarıdaki dosyaları ve SDK kütüphanelerini ekleyin,

Pin ayarlarını ve sdk_config.h içeriğini yapılandırın,

J-Link ile debug ve programlamaya başlayın.
