#include <WiFi.h>

const char* ssid = "Olgu"; 
const char* password = "olguay3k";
const int ledPin = 2; //ledin bulunduğu pin
const char* otherEspIp = "172.20.10.4"; //haberleştirilecek olan esp32 kartının ip'si

WiFiServer server(80); //sanal bir haberleşme portu

void setup() {
  pinMode(ledPin, OUTPUT); //ledin çıkış olduğunu tanımladım
  Serial.begin(115200);  //haberleşme hızı
  WiFi.begin(ssid, password); //wifi bilgileri

  while (WiFi.status() != WL_CONNECTED) { //wifi'a bağlanma şartı 
    delay(500);
    Serial.println("WiFi'a bağlanıyor...");
  }

  Serial.println("WiFi'a bağlandı.");
  Serial.print("ESP IP'si: ");
  Serial.println(WiFi.localIP()); //bağlı olan esp'nin ip'si

  server.begin();
}

void loop() {
  // İstemci olarak mesaj gönderme
  if (Serial.available() > 0) {    //eğer veri akışı olursa döndür
    String message = Serial.readStringUntil('\n'); /değişken tanıtımı
    WiFiClient client;  //espler arası bağlantı 
    if (client.connect(otherEspIp, 80)) { //esp ip'si ve portu uyuyorsa döndür
      client.println(message);  //veriyi al
      Serial.print("Gönderilen Mesaj: ");
      Serial.println(message);  monitöre yaz
    } else {
      Serial.println("Bağlantı hatası");
    }
    client.stop();
  }

  // Sunucu olarak mesaj alma
  WiFiClient client = server.available();  //haberleşme var mı kontrol
  
  if (client) {  //bağlantı varsa döndür
   
    String message = client.readStringUntil('\n');   //değişken tanımı
    message.trim();    //gelen verinin sağındaki solundaki boşlukları al
    
    if (message == "ledyak") {    //ledyak stringi gelirse döndür
      digitalWrite(ledPin, HIGH);
      Serial.println("LED YAKILDI");
    }
    else if (message == "ledsön") {    //ledsön stringi gelirse döndür
      // Eğer message değişkeni "selam" ise
      digitalWrite(ledPin, LOW);
        Serial.println("LED SÖNDÜRÜLDÜ");
     } 

else{
      Serial.print("!!!!!!!!!Tanımlanmayan veri: ");
      Serial.println(message);
    }
    
  }
  
}
