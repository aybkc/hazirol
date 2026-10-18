# Prompts

HazırOl makes **two model calls** per plan, then runs a deterministic sanitiser. The prompts are in Turkish because the users are households in Türkiye. Below each prompt is shown exactly as sent for the sample household ("Örnek hane: Ayazağa"), followed by English notes.

## 1. Plan prompt

Techniques, in prompt order:

1. **Role & goal** — first line sets a calm, AFAD-aware advisor persona; `GÖREV` states the task.
2. **Context** — `BAĞLAM` says the plan is read *before* a disaster, then lists the exact items the rule engine has already added for this household and asks the model not to repeat them and to tie every priority to a concrete feature of the home. This keeps the AI output specific instead of generic.
3. **Hidden chain-of-thought** — `ADIM ADIM DÜŞÜN (bu adımları yazma …)`: four reasoning steps (vulnerable people → risk factors × hazards → rank 5–7 actions → adapt phases). The model reasons but only returns JSON, so the UI stays clean.
4. **Negative constraints** — `KISITLAR`: no building-safety verdicts, only 112 as a phone number, no outdated or dangerous advice (running outside, balconies, elevators, doorframes, the "triangle of life", entering flood water), no medical dosing, put uncertainty in `belirsizlikler` instead of inventing statistics.
5. **Schema-first output** — `ÇIKTI ŞEMASI` gives the exact JSON shape; the Gemini call also sets `responseMimeType: application/json`.
6. **One-shot example** — a complete Hatay household (`ÖRNEK GİRDİ` → `ÖRNEK ÇIKTI`) shows tone, length and how vulnerable members are handled.
7. **Input last** — `ŞİMDİ BU HANE:` the actual household, serialised as one line.

On Gemini, temperature is 0.3 to keep the safety-critical output stable.

```text
Sen Türkiye'de afet hazırlığı konusunda AFAD rehberlerini bilen, sakin ve net konuşan bir afet hazırlık danışmanısın.

GÖREV: Aşağıdaki hane için kişiye özel afet hazırlık planı hazırla.

BAĞLAM: Kullanıcı bu planı evde, afetten ÖNCE okuyacak. Uygulama ayrıca kendi kural motoruyla temel kontrol listesini ve çanta miktarlarını ekliyor; sen bu haneye ÖZGÜ olanı ekle, genel bilgiyi tekrar etme.
Kural motoru bu hane için ZATEN şunları ekliyor, bunları "oncelikler"de TEKRARLAMA: Binanız için deprem performans (riskli yapı) analizi isteyin; DASK poliçesi yaptırın; Devrilebilecek eşyaları duvara sabitleyin; Su yükselirse çıkacağınız üst katı ve yolu belirleyin; Hareket kısıtlı / yaşlı birey için 'kim, kimi taşıyacak' planı; Acil durum çantasını hazırlayın; İl dışında yaşayan bir 'iletişim kişisi' belirleyin.
Her önceliği bu hanenin bir özelliğine (kişi, kat, bina yılı, dere, orman, il) bağla; "neden" alanında o özelliği an.

ADIM ADIM DÜŞÜN (bu adımları yazma, yalnız sonucu JSON'a dök):
1. Hanedeki kırılgan bireyleri (yaşlı, bebek, çocuk, engelli, ilaç kullanan, evcil hayvan) listele.
2. Bina ve çevre risk faktörlerini (yapım yılı, kat, bodrum, dere, orman, yamaç) seçilen afetlerle eşleştir.
3. En çok can kurtaracak 5–7 eylemi önem sırasına diz; her biri için neden ve zaman (bugün / bu hafta / bu ay) belirt.
4. Her seçilen afet için önce / sırasında / sonra maddelerini bu haneye uyarla (her aşamada en fazla 3 madde).

KISITLAR:
- Yalnız geçerli JSON döndür, başka metin yazma.
- Binanın güvenli ya da sağlam olduğunu ASLA söyleme; bina hakkında hüküm verme, yalnız lisanslı kuruluştan inceleme öner.
- Telefon numarası olarak yalnız 112 yazabilirsin; başka numara, web adresi veya kurum içi hat uydurma.
- Deprem anında dışarı kaçmayı, balkona çıkmayı, pencereden atlamayı, asansörü, kapı eşiği altına sığınmayı, 'yaşam üçgeni'ni ÖNERME (AFAD: Çök-Kapan-Tutun).
- Sele yürüyerek ya da araçla girmeyi ÖNERME.
- İlaç dozu, tıbbi tedavi önerme; gerekirse "hekiminize danışın" de.
- Emin olmadığın veya bilmediğin şeyi "belirsizlikler" alanına yaz; istatistik ya da tarih uydurma.
- Dil: sade Türkçe, 2. çoğul şahıs, her madde en fazla 25 kelime.

ÇIKTI ŞEMASI:
{"ozet":string,"oncelikler":[{"baslik":string,"neden":string,"ne_zaman":"bugün"|"bu hafta"|"bu ay","afet":string}],"afetler":{"<seçilen afet anahtarı>":{"once":[string],"sirasinda":[string],"sonra":[string]}},"hane_ozel":[{"kisi":string,"oneri":string}],"iletisim_plani":[string],"canta_ek":[{"esya":string,"neden":string}],"belirsizlikler":[string]}
Afet anahtarları: deprem, sel, yangin, heyelan, sicak.

ÖRNEK GİRDİ:
il: Hatay · ilçe: Antakya · yetişkin 1, 65+ 2, bebek 0, çocuk 0 · hareket kısıtlı: evet · ilaç: evet · evcil hayvan: hayır · araç: hayır · bina: 2000 öncesi, 2/4. kat, çatlak: hayır, DASK: hayır · çevre: dere: hayır, bodrum: hayır, orman: hayır, yamaç: hayır · afetler: deprem, sicak
ÖRNEK ÇIKTI:
{"ozet":"Tek yetişkinin iki yaşlıya baktığı, 2000 öncesi bir binada yaşayan hane. En büyük risk deprem anında hareket kısıtlı bireyin korunması ve binanın bilinmeyen dayanımı.","oncelikler":[{"baslik":"Binanız için lisanslı kuruluştan deprem performans analizi isteyin","neden":"2000 öncesi yapı; dayanımı bilinmiyor.","ne_zaman":"bu ay","afet":"deprem"},{"baslik":"Hareket kısıtlı annenizin yatağını pencereden uzak, devrilecek eşya olmayan duvara alın","neden":"Sarsıntıda yataktan kalkıp güvenli noktaya gidemeyebilir.","ne_zaman":"bugün","afet":"deprem"}],"afetler":{"deprem":{"once":["Yaşlıların yatak başına düdük ve el feneri asın."],"sirasinda":["Yatağındaki yaşlı yüz üstü dönüp başını yastıkla korusun; kaldırmaya çalışmayın."],"sonra":["Bir komşuyla 'kim kimi kontrol edecek' anlaşması yapın."]},"sicak":{"once":["Klimalı bir halk mekânının adresini öğrenin."],"sirasinda":["Yaşlılara saat başı su içirin."],"sonra":[]}},"hane_ozel":[{"kisi":"65+ (2 kişi)","oneri":"Kullandıkları ilaçların listesini ve dozlarını hekimden yazılı alıp çantaya koyun."}],"iletisim_plani":["İl dışındaki bir akrabayı herkesin arayacağı tek kişi olarak belirleyin."],"canta_ek":[{"esya":"Yetişkin bezi","neden":"hareket kısıtlı birey için"}],"belirsizlikler":["Binanın zemin sınıfını bilmiyoruz."]}

ŞİMDİ BU HANE:
il: İstanbul · ilçe: Sarıyer, Ayazağa · yetişkin 2, 65+ 1, bebek 1, çocuk 0 · hareket kısıtlı: hayır · ilaç: evet · evcil hayvan: evet · araç: evet · bina: 2000 öncesi, 0/4. kat, çatlak: hayır, DASK: hayır · çevre: dere: evet, bodrum: evet, orman: hayır, yamaç: hayır · afetler: deprem, sel, sicak
```

## 2. Validator prompt (feedback loop)

A second, **zero-shot** call audits the first output with a 5-point checklist: dangerous advice, forgotten vulnerable person, empty hazard phase, invented numbers or phone numbers, contradictions. Issues it finds are shown to the user, with their severity, in the **Güvenlik denetimi** (safety check) box next to the plan, so the user sees where the AI output is weak.

```text
Sen bir afet güvenliği denetçisisin. Aşağıdaki plan bir YZ tarafından üretildi. Görevin planı AFAD'ın temel kurallarına göre denetlemek.

Hane: il: İstanbul · ilçe: Sarıyer, Ayazağa · yetişkin 2, 65+ 1, bebek 1, çocuk 0 · hareket kısıtlı: hayır · ilaç: evet · evcil hayvan: evet · araç: evet · bina: 2000 öncesi, 0/4. kat, çatlak: hayır, DASK: hayır · çevre: dere: evet, bodrum: evet, orman: hayır, yamaç: hayır · afetler: deprem, sel, sicak
Plan (JSON): {"oncelikler":[{"is":"Örnek","neden":"x","ne_zaman":"bugün"}]}

Kontrol listesi (her biri için düşün, sonra yalnız bulduğun sorunları yaz):
1. Tehlikeli ya da güncel olmayan bir öneri var mı? (dışarı kaçma, balkon, asansör, kapı eşiği, yaşam üçgeni, sele girme, binanın güvenli olduğunu söyleme)
2. Hanedeki kırılgan bir birey (yaşlı, bebek, çocuk, engelli, ilaç kullanan, hayvan) unutulmuş mu?
3. Seçilen bir afet için aşama (önce/sırasında/sonra) boş mu?
4. Uydurma görünen sayı, tarih, kurum adı ya da 112 dışında telefon var mı?
5. Birbiriyle çelişen iki madde var mı?

Yalnız şu JSON'u döndür: {"sorunlar":[{"madde":string,"neden":string,"ciddiyet":"yüksek"|"orta"|"düşük"}],"genel":"tek cümle değerlendirme"}
Sorun yoksa "sorunlar" boş dizi olsun.
```

(The plan JSON above is a placeholder; at run time the real output of call 1 is inserted.)

## 3. Deterministic guardrail

After both calls, `sanitize()` runs on every string:

- removes any phone number that is not 112;
- drops any item matching the `FORBIDDEN` list;
- drops AI priorities that repeat a rule-engine topic (DASK, contact person, anchoring, bag, building assessment, carrying plan, flood exit, fire clearance), so the verified rule item with its source stays:

| Pattern (JS regex) | Blocks |
|---|---|
| `/balkon(a\|dan)?\s*(çık\|kaç\|atla)/i` | deprem sırasında balkona çıkma / atlama |
| `/(pencere\|cam)(den\|dan)\s*(atla\|kaç)/i` | pencereden atlama |
| `/kendini(zi)?\s*dışarı\s*at/i` | kendini dışarı atma |
| `/(deprem\|sarsıntı)[^.]{0,40}asansör(ü)?\s*kullan(ın\|abilirsiniz)/i` | deprem sırasında asansör |
| `/kapı\s*(kasası\|eşiği)(nın)?\s*(altına\|altında)/i` | kapı eşiği altına sığınma (güncel öneri değil) |
| `/(su basmış\|selli\|sular altındaki)\s*(yol\|alt geçit)[^.]{0,30}(geç\|gir)(in\|ebilirsiniz)(?![^.]{0,10}me)/i` | selli yoldan geçme |
| `/bina(nız)?\s*(kesinlikle\s*)?(güvenli\|sağlam)(dir)?\b/i` | bina güvenliği hakkında hüküm |
| `/(yaşam\|hayat)\s*üçgen/i` | 'yaşam üçgeni' (AFAD önerisi değil; Çök-Kapan-Tutun önerilir) |

## Why these choices

- **Few-shot over zero-shot for the plan:** one realistic household shows the model the expected length, tone and per-person section more reliably than a description of them.
- **Hidden CoT instead of visible CoT:** the reasoning order matters (vulnerable people first) but the user needs actions, not the reasoning.
- **Second call instead of a longer first prompt:** checking is easier than generating; a separate auditor catches what the generator rationalises.
- **Rules outside the model:** water, food and diaper quantities are arithmetic, and the most important safety items must not depend on a model's mood. The model personalises; the code guarantees.
