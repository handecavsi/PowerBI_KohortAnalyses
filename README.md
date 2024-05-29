## Kohort Analizi - Northwind Dataset

**Proje Tanımı**
Bu proje,  Northwind verisetini kullanarak Kohort Analizi gerçekleştirmektedir. Kohort Analizi, belirli bir süre boyunca belirli bir kullanıcı grubu (kohort) tarafından yapılan aktiviteleri analiz ederek kullanıcı davranışlarını ve trendleri anlamaya yardımcı olan bir tekniktir. Bu proje, kullanıcıların (müşterilerin) zaman içindeki davranışlarını ve müşteri tutma oranlarını (retention rate) hesaplamaktadır.

## Kohort Analizi Nedir?
Kohort Analizi, bir veri analiz tekniği olup, kullanıcıları veya müşterileri belirli özelliklere göre gruplandırarak (kohortlar oluşturarak) bu grupların zaman içindeki davranışlarını analiz eder. Bu analiz, özellikle müşteri yaşam döngüsü, kullanıcı etkileşimi ve müşteri sadakati gibi konularda derinlemesine içgörüler elde etmek için kullanılır.

**Anahtar Kavramlar:**
- Kohort: Ortak bir özellik veya eylemle (örneğin, belirli bir zaman diliminde ilk satın alma) tanımlanan kullanıcı grubu.
- Kohort Dönemi: Kohortların oluşturulduğu zaman dilimi (Bu proje için yıl-ay olarak oluşturuldu).
- Metriği: Analiz edilecek kullanıcı davranışları (örneğin, tekrar satın alma oranı (retention rate)).

**Veri Kümesi**
Northwind veriseti, bir dizi tablodan oluşur ve bir ticaret işletmesinin satış, müşteri, ürün ve sipariş bilgilerini içerir. Bu projede, özellikle şu tablolar kullanılmıştır:

- Customers: Müşteri bilgileri
- Orders: Sipariş bilgileri

**Kohort Analizi için kullanılan DAX formülleri:**
- Müşterilerin ilk sipariş tarihlerini orders tablosuna kolon olarak ekleme:
  
      Earliest of Month (EOM) = 
      //Sırasıyla her bir satırdaki customer_id'yi customer değişkenine atar.
      VAR customer = 'orders'[customer_id]
      RETURN
      //Her bir satırdaki customer_id için ilk sipariş tarihini getirir.
      CALCULATE(
        EOMONTH(MIN('orders'[order_date]),0),
        FILTER(
            'orders',
            'orders'[customer_id] = customer
        )
      )


- Kohort Tablosu Oluşturma:
  
      //Kohort tablosuna 0'dan 22'ye kadar 1'er artacak şekilde values ekler.
      Kohort = GENERATESERIES(0,22,1)


- Müşteri Elde Tutma Oranını Count olarak hesaplama (Retention Rate/Count):

      Customer Retention = 
      VAR CurrentMonthAfter = SELECTEDVALUE('Kohort'[Value])
      VAR CurrentFirstOrderMonth = SELECTEDVALUE(orders[Earliest of Month (EOM)])
      RETURN 
      CALCULATE(
      DISTINCTCOUNT(
        orders[customer_id]),
        FILTER(
            'orders',
            EOMONTH(orders[order_date],0) = EOMONTH(CurrentFirstOrderMonth, CurrentMonthAfter)
        )
      )


- Müşteri Elde Tutma Oranını yüzde olarak hesaplama (Retention Rate/%):

      Customer Retention % = 
      VAR KohortMonth = SELECTEDVALUE('Kohort'[Value])
      VAR CurrentFirstOrderMonth = SELECTEDVALUE(orders[Earliest of Month (EOM)])
      RETURN 
      DIVIDE(
        CALCULATE(
        DISTINCTCOUNT(
          orders[customer_id]),
          FILTER(
            'orders',
            EOMONTH(orders[order_date],0) = EOMONTH(CurrentFirstOrderMonth, KohortMonth)
          )
        ),
      DISTINCTCOUNT(orders[customer_id]))

