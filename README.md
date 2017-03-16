<h1 id="sperec">SpeRec</h1>

<p><strong>KALDI</strong> ve <strong>Java</strong> kullanarak, adim adim konusmadan metine (text-to-speech) programi gelistirme rehberi! </p>

<hr>



<h2 id="isinma">Isinma</h2>

<p>İlk olarak gerekli üçüncü parti yazılımları yükleyip çalışma ortamımızı kuracağız. Sonrasında ihtiyacımız olan verilerden bahsedip (ki burada <strong>herkes kendi ses kaydını kullanacak</strong>), ilk modelimizi oluşturacağız. Son aşamada ise -<em>Java sadece burada devreye giriyor</em>- kaydettiğimiz sesleri ekrana bastıracağız. </p>

<blockquote>
  <p><strong>On Kosullar:</strong></p>
  
  <ul>
  <li><p>Linux İşletim Sistemi</p></li>
  <li><p>Temel Linux Shell kullanımı</p></li>
  <li><p>Temel Java - Swing kodlama</p></li>
  </ul>
</blockquote>

<hr>

<p>Gerekli Yazilimlari Yükleme</p>



<h4 id="create-a-document"><i class="icon-file"></i> Create a document</h4>

<p>Bu rehber, kullanmakta olduğnuz linux dagitimin’da aşağıdaki paketlerin yüklü olduğunu varsayar. Eğer emin değilseniz aşağıdaki scripti kuruluma başlamadan önce çalıştırın: </p>

<pre><code>sudo apt-get install atlas autoconf automake git libtool svn wget zlib
</code></pre>

<p>KALDI’yı indirme: </p>

<pre><code>git clone https://github.com/kaldi-asr/kaldi
</code></pre>

<p>Bu yüklemenin ardından dosyayı indirdiğiniz konumdaki (aksini belitrmediyseniz /home/{kullanıcı adınız} klasörü) “kaldı” klasörünün içeriği:</p>

<hr>

<dl>
<dt><i class="icon-folder-open"></i>/egs</dt>
<dd>
<blockquote>
  <p>Dokumantasyonlari ile birlikte 44 farklı örnek proje. Malesef birkaçı dışında ses dosyalarını indirmek ücretli.</p>
</blockquote>
</dd>

<dt><i class="icon-folder-open"></i>/misc</dt>
<dd>
<blockquote>
  <p>Çeşitli kaynak ve pdf dosyalarını içeren, daha derine KALDI öğrenmek isteyenler için faydalı kaynaklar içeren bir klasör. Ayrıca resmi logo ve HTK’dan KALDI’ye çevirme yapan scriptler içerir. (Kabul, hiç girmedim!)</p>
</blockquote>
</dd>

<dt><i class="icon-folder-open"></i>/src</dt>
<dd>
<blockquote>
  <p>KALDI kaynak kodu</p>
</blockquote>
</dd>

<dt><i class="icon-folder-open"></i>/tools</dt>
<dd>
<blockquote>
  <p>ATLAS, SRILM gibi KALDI’nın kullandığı üçüncü parti yazılımlar. </p>
</blockquote>
</dd>

<dt><i class="icon-folder-open"></i>/windows</dt>
<dd>
<blockquote>
  <p>KALDI’yı Windows’ta kulanmak için gerekli araçlar. (Malesef, bu rehberde geçen son Windows kelimesi) </p>
</blockquote>
</dd>
</dl>

<hr>

<p>İndirme işleminden sonra gelelim kuruluma. Gerekli adımlar INSTALL dosyasında yazıyor. Tek adımda yapmak için</p>

<pre><code>      cd kaldi/tools/; make; cd ../src; ./configure; make
</code></pre>

<p>scripti yeterli. Bu adımda build yapıldığından biraz vakit alabilir, kahvenizi yenileyin!</p>

<hr>



<h2 id="veri-hazirlama">Veri Hazirlama</h2>



<h4 id="switch-to-another-document"><i class="icon-folder-open"></i> Switch to another document</h4>

<p>Verilerimizi hazirlamadan once yeni projemiz için /egs klasörünün icinde ‘Demo’ adindaki yeni klasörümüzü yaratalım.</p>

<pre><code>cd /egs; mkdir Demo; cd Demo
</code></pre>

<p>Su an Demo klasörünün içindeyiz. Modelimizin eğitim ve test verileri farklı olacağı için proje klasöründe iki ayrı klasöre ihtiyacımız var; train ve test. Bu iki klasör dışında ileride açıklayacağım ‘local’ adında bir kalsore   ve onun icinde de ‘dict’ adinda bir klasore daha ihtiyacımız var. Projemizin derli toplu olması için bunlar data adında bir klasör yaratıp, bunları data klasörünün içinde olusturalim:</p>

<pre><code>mkdir data; cd data; mkdir train; mkdir test; mkdir local; cd local; mkdir dict; cd ../../
</code></pre>

<p>Bu adımlardan sonra ‘egs’ klasörü altındaki diğer örnek projelerden almamız gereken dosya ve klasörler var. Bunu ister secim yapip copy-paste ile ister bir script ile yapabiliriz. Gerekli script;</p>

<pre><code>cd ../rm/s5;  cp -R conf exp steps utils ../../Demo/; cp cmd.sh path.sh ../../Demo
</code></pre>

<p>Verilerimiz için gerekli iskeleti kurduk, şimdi sıra içerilerini doldurmakta. <br>
Az önce verilerimizi train ve test olarak iki ayrı klasörde tutacağımızdak bahsettik. Tahmin ettiğiniz gibi train klasöründeki veriler ile modelimizi eğitip, test dosyasındaki verilerimiz ile modelimizi test edip başarısını(acçüracy) ölçeceğiz. ASR(Automated Speech Recognition) sistemlerinde model başarısını ölçmek icin kullanılan birim WER (Word Error Rate) olarak adlandırılır. WER oranı düştükçe modelimizin performansı artar. WER hakkında daha derine inmek icin: <a href="http://letmegooglethat.com/?q=Word+Error+Rate">http://letmegooglethat.com/?q=Word+Error+Rate</a></p>

<hr>

<p>Az önce yarattığımız train ve test klasörlerinin ikisinin de içerisine yaratmamız gereken <strong>metin</strong> dosyaları var. Bunların içeriği ve formatını genel olarak;</p>

<dl>
<dt><strong>wav.scp</strong></dt>
<dd>
<p><em>Format :</em> {konuşmaİD} {ses dosyasının tam dosya yolu}</p>
</dd>

<dd>
<p><em>Örnek : </em>  <code>00001    /home/user/00001.wav</code></p>

<blockquote>
  <p>Bu dosya her kayıttaki konuşma parçasını (utterance), o ses kaydının dosya yoluna bağlar. Dosya yolu mutlaka full path olmalıdır, “/home/{kullanıcıadı}/kaldi/egs/Demo/data/train/0001.wav” gibi </p>
</blockquote>
</dd>
</dl>

<hr>

<dl>
<dt><strong>text</strong></dt>
<dd>
<p><em>Format :</em> {konuşmaİD} {konuşma metni}</p>
</dd>

<dd>
<p><em>Örnek : </em>  <code>00001   fatura bilgilerim</code></p>

<blockquote>
  <p>Bu dosya her konuşmayı, o konuşmada söylenen metinle eşleştirir.</p>
</blockquote>
</dd>
</dl>

<hr>

<dl>
<dt><strong>utt2spk</strong></dt>
<dd>
<p><em>Format :</em> {konuşmaİD} {konuşmacıİD}</p>
</dd>

<dd>
<p><em>Örnek :</em>   <code>00001 00001</code></p>

<blockquote>
  <p>Bu dosya farklı konuşmacılarımız olsaydı önemli olacaktı, ama şimdilik sadece kendi sesimizle yaptığımız için İDleri aynı verelim.</p>
</blockquote>
</dd>
</dl>

<hr>

<p>Yukarıda bahsettiğim /local klasörünün içinde oluşturmamız gereken dosyalar var;</p>

<dl>
<dt><strong>corpus.txt</strong></dt>
<dd>
<p><em>Format :</em>  {konuşma metni}</p>
</dd>

<dd>
<p><em>Örnek :</em> <code>&lt;s&gt; fatura bilgilerim &lt;/s&gt;</code></p>

<blockquote>
  <p>text dosyamızdaki tüm cümleleri ARPA formatı <code>&lt;s&gt; metin &lt;/s&gt;</code> halinde ‘s’ tagları icinde sakladığımız dosya.</p>
</blockquote>
</dd>
</dl>

<hr>

<p>Yine /local içinde yarattığımız /dict içinde de lexicon adında bir dosya gerekli;</p>

<dl>
<dt><strong>lexicon</strong></dt>
<dd>
<p><em>Format :</em>  {kelime} {k e l i m e} </p>
</dd>

<dd>
<p><em>Örnek :</em> <code>gelecek  g e l e c e k</code></p>

<blockquote>
  <p>text dosyası içindeki her kelimeyi teker teker alıp, harflerin araları birer boşluklu formatta sakladığmız dosya. ‘evet  e v e t’ gibi.</p>
</blockquote>

<hr>
</dd>
</dl>
