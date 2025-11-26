<template>
  <v-container class="py-10" style="max-width: 1400px">
    <v-card class="pa-6" elevation="6" color="yellow-lighten-5">
      <h2 class="text-h5 font-weight-bold mb-4 text-yellow-darken-4">
        <v-icon icon="mdi-gold" class="mr-2" color="yellow-darken-2"></v-icon>
        Altın Taksit - Fon Getiri Hesaplama
      </h2>

      <v-row dense>
        <v-col cols="12" md="9">
          <v-form v-model="formValid">
            <v-row dense>
              <v-col cols="12" sm="6">
                <v-text-field color="red" label="Peşin Altın Fiyatı (TL)" v-model.number="cashPrice" type="number"
                  :rules="positiveNumberRule" required />
              </v-col>

              <v-col cols="12" sm="6">
                <v-text-field color="red" label="Taksitli Fiyat (TL)" v-model.number="installmentPrice" type="number"
                  :rules="positiveNumberRule" required />
              </v-col>

              <v-col cols="12" sm="4">
                <v-text-field color="red" label="Taksit Sayısı" v-model.number="installmentCount" type="number"
                  :rules="installmentCountRule" required />
              </v-col>

              <v-col cols="12" sm="4">
                <v-text-field color="red" label="Bonus (TL) — Tek Seferlik" v-model.number="bonus" type="number"
                  :rules="nonNegativeNumberRule" hint="Kredi kartı bonusu" required />
              </v-col>

              <v-col cols="12" sm="4">
                <v-text-field color="red" label="Günlük Getiri (%)" v-model.number="dailyReturnPercent" type="number" suffix="%"
                  :rules="percentRule" step="0.01" required />
              </v-col>

              <v-col cols="12" sm="4">
                <v-text-field color="red" label="Satın Alma Tarihi" type="date" v-model="purchaseDate" :rules="dateRule" required />
              </v-col>

              <v-col cols="12" sm="4">
                <v-text-field color="red" label="Ekstre Kesim Tarihi" type="date" v-model="cutoffDate"
                  @change="updateAutoPaymentDate" :rules="cutoffDateRule" hint="Kredi kartı ekstre kesim tarihi"
                  required />
              </v-col>

              <v-col cols="12" sm="4">
                <v-text-field color="red" label="Fatura Ödeme Tarihi (ilk ödeme)" type="date" v-model="firstPaymentDate"
                  :rules="firstPaymentDateRule" hint="İlk taksit ödeme tarihi" required />
              </v-col>

              <v-col cols="12" sm="4" offset="4" class="d-flex align-center">
                <v-btn color="red-darken-1" @click="calculate" :disabled="!formValid" size="large" elevation="2"
                  block>
                  <v-icon icon="mdi-calculator" class="mr-2"></v-icon>
                  Hesapla
                </v-btn>
              </v-col>
            </v-row>
          </v-form>
        </v-col>

        <v-col cols="12" md="3">
          <div class="text-subtitle-1 font-weight-bold mb-3 text-amber-darken-3">Kredi Kartları</div>
          <div class="d-flex flex-column ga-3">
            <v-card v-for="card in creditCards" :key="card.id" @click="selectCard(card)" :color="card.color"
              :class="selectedCard?.id === card.id ? 'card-selected' : 'card-unselected'" class="pa-3 cursor-pointer"
              :elevation="selectedCard?.id === card.id ? 8 : 2" hover>
              <div class="d-flex align-center mb-2">
                <v-icon :icon="card.icon" size="large" color="white" class="mr-2"></v-icon>
                <div class="text-white font-weight-bold text-body-2">{{ card.name }}</div>
              </div>
              <v-divider class="my-2 opacity-50" color="white"></v-divider>
              <div class="text-white text-caption">
                <div class="d-flex justify-space-between mb-1">
                  <span>Kesim:</span>
                  <span class="font-weight-bold">{{ card.cutoffDay }}/{{ card.cutoffMonth }}</span>
                </div>
                <div class="d-flex justify-space-between">
                  <span>Ödeme:</span>
                  <span class="font-weight-bold">{{ card.paymentDay }}/{{ card.paymentMonth }}</span>
                </div>
              </div>
            </v-card>
          </div>
        </v-col>
      </v-row>

      <v-divider class="my-4"></v-divider>

      <!-- Maksimum Taksitli Fiyat - Hesapla butonundan bağımsız -->
      <div v-if="maxInstallmentPrice > 0">
        <h3 class="text-h6 font-weight-bold mb-3 text-indigo-darken-3">Kârlılık Analizi</h3>
        <v-card class="pa-4 mb-4" color="indigo-lighten-4" elevation="3">
          <div class="text-h6 mb-2 text-indigo-darken-4">
            <v-icon icon="mdi-chart-line-variant" class="mr-2" color="indigo-darken-2"></v-icon>
            <strong>Kârlı olmak için maksimum taksitli fiyat:</strong>
          </div>
          <div class="text-h4 font-weight-bold text-indigo-darken-3 my-3">
            {{ format(maxInstallmentPrice) }} TL
          </div>
          <div class="text-caption mt-2 text-indigo-darken-2">
            Bu değer: Peşin Fiyat ({{ format(cashPrice) }}) + Ana Fon Getirisi + Bonus + Bonus Getirisi
          </div>
        </v-card>
      </div>

      <!-- TABLE -->
      <div v-if="rows.length">
        <h3 class="text-h6 font-weight-bold mb-3 text-amber-darken-3">Fon Çalışma Tablosu</h3>

        <v-data-table dense hide-default-footer class="elevation-2">
          <thead>
            <tr>
              <th>Taksit No</th>
              <th>Başlangıç</th>
              <th>Bitiş (Ödeme)</th>
              <th>Fon Gün</th>
              <th>Ana Fon (TL)</th>
              <th>Ana Fon Getiri (TL)</th>
              <th>Bonus Fon (TL)</th>
              <th>Bonus Gün</th>
              <th>Bonus Getiri (TL)</th>
              <th>Ödenecek Taksit (TL)</th>
              <th>Ödeme Tarihi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(r, idx) in rows" :key="idx">
              <td>{{ r.installmentIndex }}</td>
              <td>{{ r.start }}</td>
              <td>{{ r.end }}</td>
              <td>{{ r.fundDays }}</td>
              <td>{{ format(r.mainFundAmount) }}</td>
              <td>{{ format(r.mainFundGain) }}</td>
              <td>{{ format(r.bonusFundAmount) }}</td>
              <td>{{ r.bonusDays }}</td>
              <td>{{ format(r.bonusGain) }}</td>
              <td>{{ format(r.installmentPay) }}</td>
              <td>{{ r.payDate }}</td>
            </tr>
          </tbody>
        </v-data-table>

        <v-divider class="my-4"></v-divider>

        <v-row>
          <v-col cols="12" md="6">
            <v-card class="pa-3" color="amber-lighten-4" elevation="2">
              <div class="text-amber-darken-4 mb-2">
                <v-icon icon="mdi-cash-multiple" class="mr-2" color="amber-darken-2"></v-icon>
                <strong>Kazanç Detayları</strong>
              </div>
              <div><strong>Toplam Ana Fon Getirisi:</strong> {{ format(totalMainFundGain) }} TL</div>
              <div><strong>Toplam Bonus Fon Ana Para:</strong> {{ format(totalBonusPrincipal) }} TL</div>
              <div><strong>Toplam Bonus Fon Getirisi:</strong> {{ format(totalBonusGain) }} TL</div>
              <div class="mt-2 text-h6"><strong>Taksitli elde edilen toplam kazanç:</strong> {{ format(totalGain) }} TL
              </div>
            </v-card>
          </v-col>

          <v-col cols="12" md="6">
            <v-card class="pa-3" :color="netProfit >= 0 ? 'green-lighten-4' : 'red-lighten-4'" elevation="2">
              <div class="mb-2" :class="netProfit >= 0 ? 'text-green-darken-4' : 'text-red-darken-4'">
                <v-icon :icon="netProfit >= 0 ? 'mdi-thumb-up' : 'mdi-thumb-down'" class="mr-2"></v-icon>
                <strong>Net Kar / Zarar:</strong>
              </div>
              <div class="text-h4 font-weight-bold"
                :class="netProfit >= 0 ? 'text-green-darken-3' : 'text-red-darken-3'">
                {{ format(netProfit) }} TL
              </div>
            </v-card>
          </v-col>
        </v-row>
      </div>
    </v-card>
  </v-container>
</template>

<script setup lang="ts">
/**
 * ALTIN TAKSİT - FON GETİRİ HESAPLAMA
 * 
 * Bu uygulama, altını taksitle mi almalı yoksa peşin mi almalı sorusunu yanıtlar.
 * Mantık: Peşin parayı fona yatırıp, taksitleri fondan ödeyerek kar/zarar analizi yapar.
 * 
 * Senaryo:
 * 1. Altını peşin alacak parayı fona yatırıyoruz
 * 2. Her ay fon getirisi kazanıyoruz
 * 3. Taksit ödemelerini fondan yapıyoruz
 * 4. Kredi kartı bonusu varsa, o da fonda çalışıyor
 * 5. Sonunda toplam kazancımızı, taksit farkı ile karşılaştırıyoruz
 * 
 * Eğer toplam kazanç > taksit farkı ise → Taksitli almak karlı
 * Eğer toplam kazanç < taksit farkı ise → Peşin almak daha mantıklı
 */

import { ref, computed, watch } from 'vue'
import moment from 'moment'

/* ========================================
   KART TANIMLARI
   ======================================== */
/**
 * Kredi kartı bilgilerini tutar.
 * Her kartın ekstre kesim ve ödeme tarihleri farklıdır.
 */
interface CreditCard {
  id: string          // Benzersiz kimlik
  name: string        // Kart adı (örn: Ziraat Bankası)
  cutoffDay: number   // Ekstre kesim günü (ayın kaçıncı günü)
  cutoffMonth: number // Ekstre kesim ayı
  paymentDay: number  // Ödeme günü (ayın kaçıncı günü)
  paymentMonth: number // Ödeme ayı
  icon: string        // Material Design ikonu
  color: string       // Vuetify renk kodu
}

/**
 * Önceden tanımlanmış kredi kartları
 * Kullanıcı bu kartları seçerek otomatik tarih ayarlaması yapabilir
 */
const creditCards: CreditCard[] = [
  {
    id: 'ziraat',
    name: 'Ziraat Bankası',
    cutoffDay: 5,        // Her ayın 5'inde ekstre kesilir
    cutoffMonth: 12,     // Aralık ayı (statik tanım için)
    paymentDay: 17,      // Her ayın 17'sinde ödeme yapılır
    paymentMonth: 12,    // Aralık ayı (statik tanım için)
    icon: 'mdi-credit-card-check',
    color: 'red darken-4'
  },
  {
    id: 'akbank',
    name: 'Akbank',
    cutoffDay: 4,
    cutoffMonth: 12,
    paymentDay: 14,
    paymentMonth: 12,
    icon: 'mdi-credit-card-wireless',
    color: 'deep-orange darken-2'
  },
  {
    id: 'garanti',
    name: 'Garanti BBVA',
    cutoffDay: 2,
    cutoffMonth: 12,
    paymentDay: 12,
    paymentMonth: 12,
    icon: 'mdi-credit-card-plus',
    color: 'green darken-3'
  },
  {
    id: 'dummy',
    name: 'Dummy Card',
    cutoffDay: 1,
    cutoffMonth: 12,
    paymentDay: 10,
    paymentMonth: 12,
    icon: 'mdi-credit-card',
    color: 'blue-grey darken-1'
  }
]

// Seçili kredi kartı (null = henüz seçilmedi)
const selectedCard = ref<CreditCard | null>(null)

/* ========================================
   DURUM DEĞİŞKENLERİ (STATE)
   ======================================== */
// Kullanıcı girdileri
const cashPrice = ref<number>(55000)              // Altının peşin fiyatı (TL)
const installmentPrice = ref<number>(58000)       // Altının taksitli fiyatı (TL)
const installmentCount = ref<number>(3)           // Taksit sayısı (ay)
const dailyReturnPercent = ref<number>(0.08)      // Fon günlük getiri oranı (%)
const bonus = ref<number>(500)                    // Kredi kartı bonusu (TL) - tek seferlik

const purchaseDate = ref<string>(moment().format('YYYY-MM-DD'))  // Satın alma tarihi (bugün)
const cutoffDate = ref<string>('2025-12-02')                      // Ekstre kesim tarihi
const firstPaymentDate = ref<string>('2025-12-12')                // İlk taksit ödeme tarihi

// Hesaplama sonuçları
const rows = ref<any[]>([])                       // Tablo satırları (ayrıntılı hesaplamalar)
const totalMainFundGain = ref<number>(0)          // Toplam ana fon getirisi (TL)
const totalBonusGain = ref<number>(0)             // Toplam bonus fon getirisi (TL)
const totalBonusPrincipal = ref<number>(0)        // Toplam bonus ana para (TL)
const totalGain = ref<number>(0)                  // Toplam kazancımız (Ana + Bonus + Bonus Getiri)
const netProfit = ref<number>(0)                  // Net kar/zarar (Toplam Kazanç - Taksit Farkı)

// Form doğrulama durumu
const formValid = ref(false)  // true = tüm alanlar geçerli, false = hatalı alan var

/**
 * Sayıyı Türkçe formatında gösterir (1.234,56 TL)
 */
const format = (n: number) =>
  n.toLocaleString('tr-TR', { minimumFractionDigits: 2, maximumFractionDigits: 2 })

/* ========================================
   KART SEÇİM FONKSİYONU
   ======================================== */
/**
 * Kullanıcı bir kredi kartı seçtiğinde çağrılır.
 * - Kartın ekstre kesim ve ödeme tarihlerini otomatik ayarlar
 * - Satın alma tarihi değişmez (bugünün tarihi olarak kalır)
 * - Form doluysa otomatik hesaplama yapar
 */
const selectCard = (card: CreditCard) => {
  selectedCard.value = card

  // Sadece ekstre kesim ve ödeme tarihlerini otomatik ayarla
  const currentYear = 2025
  cutoffDate.value = `${currentYear}-${String(card.cutoffMonth).padStart(2, '0')}-${String(card.cutoffDay).padStart(2, '0')}`
  firstPaymentDate.value = `${currentYear}-${String(card.paymentMonth).padStart(2, '0')}-${String(card.paymentDay).padStart(2, '0')}`

  // Form geçerliyse otomatik hesapla (kullanıcı tekrar hesaplaya tıklamasın)
  if (formValid.value) {
    calculate()
  }
}

/* ========================================
   FORM DOĞRULAMA KURALLARI
   ======================================== */
// Temel zorunlu alan kontrolü
const requiredRule = [(v: any) => !!v || 'Bu alan zorunludur']

// Pozitif sayı kontrolü (0'dan büyük olmalı)
const positiveNumberRule = [
  (v: any) => v !== null && v !== undefined && v !== '' || 'Bu alan zorunludur',
  (v: any) => v > 0 || 'Pozitif bir değer giriniz'
]

// Negatif olmayan sayı kontrolü (0 veya pozitif, bonus için)
const nonNegativeNumberRule = [
  (v: any) => v !== null && v !== undefined && v !== '' || 'Bu alan zorunludur',
  (v: any) => v >= 0 || 'Negatif olamaz'
]

// Taksit sayısı kontrolü (1-12 arası tam sayı)
const installmentCountRule = [
  (v: any) => !!v || 'Bu alan zorunludur',
  (v: any) => v > 0 || 'Pozitif bir değer giriniz',
  (v: any) => Number.isInteger(Number(v)) || 'Tam sayı giriniz',
  (v: any) => v <= 12 || 'Maksimum 12 taksit olabilir'
]

// Yüzde kontrolü (0-100 arası)
const percentRule = [
  (v: any) => v !== null && v !== undefined && v !== '' || 'Bu alan zorunludur',
  (v: any) => v >= 0 || 'Negatif olamaz',
  (v: any) => v <= 100 || 'Maksimum %100 olabilir'
]

// Tarih kontrolü
const dateRule = [(v: any) => !!v || 'Tarih seçiniz']

/**
 * Ekstre kesim tarihi doğrulama
 * - Zorunlu alan
 * - Ödeme tarihinden önce olmalı
 */
const cutoffDateRule = computed(() => [
  (v: any) => !!v || 'Ekstre kesim tarihi zorunludur',
  (v: any) => {
    if (!v || !firstPaymentDate.value) return true
    return moment(v).isBefore(moment(firstPaymentDate.value)) || 'Ekstre kesim tarihi, ödeme tarihinden önce olmalı'
  }
])

/**
 * İlk ödeme tarihi doğrulama
 * - Zorunlu alan
 * - Ekstre kesim tarihinden sonra olmalı
 */
const firstPaymentDateRule = computed(() => [
  (v: any) => !!v || 'Ödeme tarihi zorunludur',
  (v: any) => {
    if (!v || !cutoffDate.value) return true
    return moment(v).isAfter(moment(cutoffDate.value)) || 'Ödeme tarihi, ekstre kesim tarihinden sonra olmalı'
  }
])

/* ========================================
   YARDIMCI FONKSİYONLAR
   ======================================== */

/**
 * İki tarih arası fon getirisini hesaplar
 * Formül: Ana Para * (1 + Günlük Oran)^Gün Sayısı - Ana Para
 * 
 * @param principal - Ana para (TL)
 * @param startDate - Başlangıç tarihi (dahil)
 * @param endDate - Bitiş tarihi (hariç)
 * @param dailyRate - Günlük getiri oranı (ondalık: 0.08 = %0.08)
 * @returns Getiri miktarı (TL)
 */
function calculateFundGain(principal: number, startDate: moment.Moment, endDate: moment.Moment, dailyRate: number): number {
  const days = Math.max(0, endDate.diff(startDate, 'days'))  // Gün sayısı
  return principal * (Math.pow(1 + dailyRate, days) - 1)      // Sadece getiri
}

/**
 * Fonu günceller (ana para + getiri)
 * Formül: Ana Para * (1 + Günlük Oran)^Gün Sayısı
 * 
 * @param principal - Ana para (TL)
 * @param startDate - Başlangıç tarihi
 * @param endDate - Bitiş tarihi
 * @param dailyRate - Günlük getiri oranı
 * @returns Yeni fon değeri (ana para + getiri)
 */
function applyGain(principal: number, startDate: moment.Moment, endDate: moment.Moment, dailyRate: number): number {
  const days = Math.max(0, endDate.diff(startDate, 'days'))
  return principal * Math.pow(1 + dailyRate, days)  // Ana para + getiri
}

/**
 * Ekstre kesim tarihine göre ödeme tarihini otomatik ayarlar
 * Kural: Ekstre kesim + 10 gün = Ödeme tarihi
 */
function updateAutoPaymentDate() {
  if (!cutoffDate.value) return
  firstPaymentDate.value = moment(cutoffDate.value, 'YYYY-MM-DD').add(10, 'days').format('YYYY-MM-DD')
}

/**
 * Taksit ödeme tarihlerini oluşturur
 * 
 * @param firstPayStr - İlk ödeme tarihi (YYYY-MM-DD)
 * @param count - Taksit sayısı
 * @param shiftMonths - Ay kaydırma (satın alma kesim gününde veya sonrasında ise +1)
 * @returns Ödeme tarihleri dizisi
 * 
 * Örnek: firstPay=15.12.2025, count=3, shift=0 -> [15.12.2025, 15.01.2026, 15.02.2026]
 */
function buildPaymentDates(firstPayStr: string, count: number, shiftMonths = 0): moment.Moment[] {
  const arr: moment.Moment[] = []
  let d = moment(firstPayStr, 'YYYY-MM-DD').clone().add(shiftMonths, 'month')
  arr.push(d.clone())
  for (let i = 1; i < count; i++) {
    d = d.clone().add(1, 'month')  // Her seferinde 1 ay ekle (gün sabit kalır)
    arr.push(d.clone())
  }
  return arr
}

/* ========================================
   COMPUTED: MAKSİMUM TAKSİTLİ FİYAT (BİNARY SEARCH)
   ======================================== */
/**
 * Kar/zarar = 0 olacak şekilde maksimum taksitli fiyatı hesaplar.
 * 
 * Kullanıcının girdiği taksitli fiyattan bağımsızdır.
 * Sadece peşin fiyat, taksit sayısı, tarihler ve getiri oranına bakar.
 * 
 * Mantık:
 * - Kullanıcı bu fiyata kadar taksitle alsaydı ne kadar kazanacaktı?
 * - Bu kazancı tam olarak (peşin-taksit) farkına eşit yapan fiyat nedir?
 * 
 * Yöntem: Binary Search (ikili arama)
 * - Min fiyat = peşin fiyat
 * - Max fiyat = peşin * 1.5
 * - Her iterasyonda ortadaki fiyatı test eder
 * - 50 iterasyonda 0.01 TL hassasiyetle sonuç bulur
 * 
 * @returns Maksimum taksitli fiyat (TL)
 */
const maxInstallmentPrice = computed<number>(() => {
  // Gerekli alanlar doluysa hesapla (taksitli fiyat hariç)
  if (!cashPrice.value || !installmentCount.value || !purchaseDate.value || !firstPaymentDate.value || !dailyReturnPercent.value) {
    return 0
  }

  try {
    const purchase = moment(purchaseDate.value, 'YYYY-MM-DD')
    const cutoff = cutoffDate.value ? moment(cutoffDate.value, 'YYYY-MM-DD') : null
    const shiftMonths = cutoff && purchase.isSameOrAfter(cutoff, 'day') ? 1 : 0
    const paymentDates = buildPaymentDates(firstPaymentDate.value, installmentCount.value, shiftMonths)
    const dailyRate = dailyReturnPercent.value / 100

    // Dönemleri oluştur
    const periods: { start: moment.Moment; end: moment.Moment }[] = []
    let start = purchase.clone()
    for (let i = 0; i < paymentDates.length; i++) {
      const end = paymentDates[i].clone()
      periods.push({ start: start.clone(), end: end.clone() })
      start = end.clone().add(1, 'day')
    }

    // Bonus mantığı
    const bonusCredit = purchase.clone().add(1, 'month').startOf('month')
    let idxPaymentAfterBonusCredit = -1
    for (let j = 0; j < paymentDates.length; j++) {
      if (paymentDates[j].isSameOrAfter(bonusCredit, 'day')) {
        idxPaymentAfterBonusCredit = j
        break
      }
    }
    const bonusStartWorkingPeriodIndex = idxPaymentAfterBonusCredit >= 0 ? idxPaymentAfterBonusCredit + 1 : -1

    // Binary search: Net kar/zarar = 0 olan taksitli fiyatı bul
    let minPrice = cashPrice.value         // Alt sınır (peşin fiyat)
    let maxPrice = cashPrice.value * 1.5   // Üst sınır (peşin * 1.5)
    let bestPrice = cashPrice.value        // En iyi tahmin

    // 50 iterasyon yeterli (2^50 = 1 katrilyon seçenek arasından bulur)
    for (let iteration = 0; iteration < 50; iteration++) {
      const testPrice = (minPrice + maxPrice) / 2           // Orta nokta
      const testInstallmentPay = testPrice / installmentCount.value  // Aylık taksit

      let testMainFund = cashPrice.value
      let testBonusFund = 0
      let testTotalMainGain = 0
      let testTotalBonusGain = 0
      let testBonusPrincipal = 0

      for (let i = 0; i < periods.length; i++) {
        const p = periods[i]

        // Ana fon getirisi
        const mainGain = calculateFundGain(testMainFund, p.start, p.end, dailyRate)
        testTotalMainGain += mainGain

        // Bonus aktif mi?
        if (i === bonusStartWorkingPeriodIndex && bonus.value > 0 && testBonusFund === 0) {
          testBonusFund = bonus.value
          testBonusPrincipal = bonus.value
        }

        // Bonus getirisi
        if (testBonusFund > 0) {
          const bonusGain = calculateFundGain(testBonusFund, p.start, p.end, dailyRate)
          testTotalBonusGain += bonusGain
          testBonusFund = applyGain(testBonusFund, p.start, p.end, dailyRate)
        }

        // Ana fonu güncelle
        testMainFund = applyGain(testMainFund, p.start, p.end, dailyRate) - testInstallmentPay
      }

      const testTotalGain = testTotalMainGain + testBonusPrincipal + testTotalBonusGain
      const testNetProfit = testTotalGain - (testPrice - cashPrice.value)

      // Net kar/zarar = 0'a yakınsadıysak dur
      if (Math.abs(testNetProfit) < 0.01) {
        bestPrice = testPrice
        break
      }

      // Net kar pozitifse, fiyatı artırabilir
      if (testNetProfit > 0) {
        minPrice = testPrice
      } else {
        maxPrice = testPrice
      }

      bestPrice = testPrice
    }

    return bestPrice
  } catch (error) {
    return 0
  }
})

/* ========================================
   ANA HESAPLAMA FONKSİYONU
   ======================================== */
/**
 * Kullanıcının girdiği taksitli fiyata göre detaylı kar/zarar analizi yapar.
 * 
 * Adımlar:
 * 1. Satın alma tarihi ve ödeme tarihlerine göre dönemleri oluşturur
 * 2. Her dönem için:
 *    - Ana fon getirisini hesaplar
 *    - Bonus aktifse bonus getirilerini hesaplar
 *    - Taksiti ana fondan öder
 * 3. Tablo satırlarını oluşturur
 * 4. Toplam kazanç ve net kar/zarar hesaplar
 * 
 * Bonus Mantığı:
 * - Bonus, satın alma tarihinden sonraki ayın 1'inde ekstreden kesilir
 * - O ödeme tarihinde ödenir
 * - Bir sonraki dönemin başında fonda çalışmaya başlar
 */
function calculate() {
  // Önceki sonuçları temizle
  rows.value = []
  totalMainFundGain.value = 0
  totalBonusGain.value = 0
  totalBonusPrincipal.value = 0
  totalGain.value = 0
  netProfit.value = 0

  // Zorunlu alanlar kontrolü
  if (!purchaseDate.value || !firstPaymentDate.value) {
    alert('Lütfen Satın Alma ve İlk Ödeme tarihlerini girin.')
    return
  }

  const purchase = moment(purchaseDate.value, 'YYYY-MM-DD')
  const cutoff = cutoffDate.value ? moment(cutoffDate.value, 'YYYY-MM-DD') : null

  /**
   * ÖNEMLİ: Ekstre Kesim Tarihi Mantığı
   * 
   * Eğer satın alma tarihi, ekstre kesim tarihinde veya sonrasında ise:
   * → Bu alışveriş bir sonraki ekstrede görünür
   * → İlk ödeme 1 ay sonra kaydırılır
   * 
   * Örnek:
   * - Ekstre kesim: 5 Aralık
   * - Satın alma: 5 Aralık veya sonrası
   * - İlk ödeme tarihi: 17 Aralık (veri girişi)
   * - Gerçek ilk ödeme: 17 Ocak (1 ay kaydırıldı)
   */
  const shiftMonths = cutoff && purchase.isSameOrAfter(cutoff, 'day') ? 1 : 0
  const paymentDates = buildPaymentDates(firstPaymentDate.value, installmentCount.value, shiftMonths)

  const dailyRate = dailyReturnPercent.value / 100      // Yüzdeyi ondalığa çevir
  const installmentPay = installmentPrice.value / installmentCount.value  // Aylık taksit

  /**
   * Dönemleri Oluştur (Periods)
   * 
   * Her dönem = Bir ödeme periyodu
   * - Başlangıç: Önceki dönemin bitişi + 1 gün (ilk dönem için satın alma tarihi)
   * - Bitiş: Ödeme tarihi
   * 
   * Örnek (3 taksit):
   * Dönem 1: 05.12.2025 → 17.12.2025 (satın alma → ilk ödeme)
   * Dönem 2: 18.12.2025 → 17.01.2026 (ilk ödeme + 1 gün → ikinci ödeme)
   * Dönem 3: 18.01.2026 → 17.02.2026 (ikinci ödeme + 1 gün → üçüncü ödeme)
   */
  const periods: { start: moment.Moment; end: moment.Moment; payDate: moment.Moment }[] = []
  let start = purchase.clone()
  for (let i = 0; i < paymentDates.length; i++) {
    const end = paymentDates[i].clone()
    periods.push({ start: start.clone(), end: end.clone(), payDate: end.clone() })
    start = end.clone().add(1, 'day')  // Bir sonraki dönem, bu ödemeden 1 gün sonra başlar
  }

  /**
   * Bonus Mantığı
   * 
   * Örnek: Satın alma 15.01.2025
   * 1. Bonus ekstreden kesilme: 01.02.2025 (bir sonraki ayın 1'i)
   * 2. Bonus ödemesi: İlk ödeme tarihi >= 01.02.2025 olan ödeme
   * 3. Bonus fonda çalışmaya başlar: O ödemeden sonraki dönemin başında
   * 
   * Neden böyle?
   * - Kredi kartı bonusu, kullanıldığı ayın ekstresinde görünür
   * - Ekstre kesiminden sonra ödeme yapılır
   * - Ödeme yapıldıktan sonra elimize geçer ve fona koyabiliriz
   */
  const bonusCredit = purchase.clone().add(1, 'month').startOf('month')  // Bonus ekstreden kesilme tarihi

  // Bonus ödemesinden sonraki ilk ödemeyi bul
  let idxPaymentAfterBonusCredit = -1
  for (let j = 0; j < paymentDates.length; j++) {
    if (paymentDates[j].isSameOrAfter(bonusCredit, 'day')) {
      idxPaymentAfterBonusCredit = j
      break
    }
  }

  // Bonus fonda çalışmaya başlayan dönem index'i
  const bonusStartWorkingPeriodIndex = idxPaymentAfterBonusCredit >= 0 ? idxPaymentAfterBonusCredit + 1 : -1

  /**
   * Dönemler Üzerinde İterasyon
   * 
   * Her dönem için:
   * 1. Ana fon getirisi hesapla
   * 2. Bonus aktifse bonus getirisi hesapla
   * 3. Tablo satırı oluştur
   * 4. Ana fondan taksit öde
   * 5. Bir sonraki döneme geç
   */
  let currentMainFund = cashPrice.value   // Ana fon başlangıç değeri = Peşin fiyat
  let currentBonusFund = 0                // Bonus başlangıçta 0, ilgili dönemde aktif olur
  let installmentCounter = 0              // Taksit numarası

  for (let i = 0; i < periods.length; i++) {
    const p = periods[i]
    const startStr = p.start.format('DD.MM.YYYY')
    const endStr = p.end.format('DD.MM.YYYY')

    // Gün sayısı hesabı: Başlangıç dahil, bitiş (ödeme günü) hariç
    const fundDays = Math.max(0, p.end.diff(p.start, 'days'))

    installmentCounter++
    const installmentIndex = installmentCounter

    // Ana fon getirisi hesapla (sadece getiri, ana para ayrı)
    const mainFundGain = currentMainFund * (Math.pow(1 + dailyRate, fundDays) - 1)

    // Bonus fon durumu
    let bonusFundAmount = currentBonusFund
    let bonusDays = 0
    let bonusGain = 0

    // Bu dönemde bonus aktif mi?
    if (i === bonusStartWorkingPeriodIndex && bonus.value > 0 && currentBonusFund === 0) {
      // Bonus bu dönemin başında fona ekleniyor
      currentBonusFund = bonus.value
      bonusFundAmount = currentBonusFund
      totalBonusPrincipal.value = bonus.value
    }

    // Bonus varsa getirisi hesapla
    if (currentBonusFund > 0) {
      bonusDays = fundDays
      bonusGain = currentBonusFund * (Math.pow(1 + dailyRate, fundDays) - 1)
    }

    // Tablo satırı oluştur
    rows.value.push({
      installmentIndex,        // Taksit no
      start: startStr,         // Dönem başlangıcı
      end: endStr,             // Dönem bitişi (ödeme günü)
      fundDays,                // Fon çalışma gün sayısı
      mainFundAmount: currentMainFund,    // Ana fon miktarı
      mainFundGain,                       // Ana fon getirisi
      bonusFundAmount,                    // Bonus fon miktarı
      bonusDays,                          // Bonus çalışma gün sayısı
      bonusGain,                          // Bonus getirisi
      installmentPay,                     // Ödenecek taksit
      payDate: p.payDate.format('DD.MM.YYYY'),  // Ödeme tarihi
    })

    // Toplamları güncelle
    totalMainFundGain.value += mainFundGain
    totalBonusGain.value += bonusGain

    // Bir sonraki dönem için fonları güncelle
    // Ana fon: Getiri ekle, taksit öde
    let mainFundAfter = currentMainFund * Math.pow(1 + dailyRate, fundDays) - installmentPay

    // Bonus fon: Sadece getiri ekle (taksit ana fondan ödenir, bonusa dokunulmaz)
    let bonusFundAfter = currentBonusFund > 0 ? currentBonusFund * Math.pow(1 + dailyRate, fundDays) : 0

    currentMainFund = mainFundAfter
    currentBonusFund = bonusFundAfter
  }

  /**
   * Toplam Sonuçlar
   * 
   * Toplam Kazanç = Ana Fon Getirisi + Bonus Ana Para + Bonus Getirisi
   * Net Kar/Zarar = Toplam Kazanç - (Taksitli Fiyat - Peşin Fiyat)
   * 
   * Pozitif → Taksitli almak karlı
   * Negatif → Peşin almak daha mantıklı
   * Sıfır → Fark etmez
   */
  totalGain.value = totalMainFundGain.value + totalBonusPrincipal.value + totalBonusGain.value
  netProfit.value = totalGain.value - (installmentPrice.value - cashPrice.value)
}
</script>

<style scoped>
/* ========================================
   CSS STIL TANIMLARI
   ======================================== */

/* Başarı mesajları için yeşil renk */
.text-success {
  color: #2e7d32;
}

/* Hata mesajları için kırmızı renk */
.text-error {
  color: #c62828;
}

/* Tıklanabilir elementler için imleç */
.cursor-pointer {
  cursor: pointer;
}

/* Seçili kart stili (büyütme ve vurgulama) */
.card-selected {
  transform: scale(1.05);
  /* 5% büyütme */
  transition: all 0.3s ease;
  /* Yumuşak geçiş */
  border: 2px solid white;
  /* Beyaz kenarlık */
}

/* Seçili olmayan kart stili (hafif soluk) */
.card-unselected {
  opacity: 0.85;
  /* Hafif saydam */
  transition: all 0.3s ease;
  /* Yumuşak geçiş */
}

/* Seçili olmayan karta hover (üzerine gelindiğinde) */
.card-unselected:hover {
  opacity: 1;
  /* Tam opak */
  transform: scale(1.02);
  /* Hafif büyütme */
}
</style>
