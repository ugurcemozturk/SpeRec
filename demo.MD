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

<p>Az önce yarattığımız train ve test klasörlerinin ikisinin de içerisine yaratmamız gereken metin dosyaları var. Bunların içeriği ve formatını genel olarak;</p>

<hr>

<dl>
<dt><strong>wav.scp</strong></dt>
<dd>
<p>{konuşmaİD} {ses dosyasının tam dosya yolu}</p>

<blockquote>
  <p>Bu dosya her kayıttaki konuşma parçasını (utterance), o ses kaydının dosya yoluna bağlar. Dosya yolu mutlaka full path olmalıdır, “/home/{kullanıcı adı}/kaldi/egs/Demo/data/train/0001.wav” gibi </p>
</blockquote>
</dd>
</dl>

<hr>

<dl>
<dt><strong>text</strong></dt>
<dd>
<p>{konuşmaİD} {konuşma metni}</p>

<blockquote>
  <p>Bu dosya her konuşmayı, o konuşmada söylenen metinle eşleştirir.</p>
</blockquote>
</dd>
</dl>

<hr>

<dl>
<dt><strong>utt2spk</strong></dt>
<dd>
<p>{konuşmaİD} {konuşmacıİD}</p>

<blockquote>
  
</blockquote>
</dd>
</dl>

<hr>

<p>Yukarıda bahsettiğim /local klasörünün içinde oluşturmamız gereken dosyalar var;</p>

<hr>

<dl>
<dt><strong>corpus.txt</strong></dt>
<dd>
<p>{konuşma metni}</p>

<blockquote>
  <p>text dosyamızdaki tüm cümleleri ARPA formatında sakladığımız dosya.</p>
</blockquote>
</dd>
</dl>

<hr>

<p>Yine /local içinde yarattığımız /dict içinde de lexicon adında bir dosya gerekli;</p>

<dl>
<dt><strong>lexicon</strong></dt>
<dd>
<p>{kelime}{k e l i m e} </p>

<blockquote>
  <p>text dosyası içindeki her kelimeyi teker teker alıp, harflerin araları birer boşluklu formatta sakladığmız dosya. ‘evet e v e t’ gibi.</p>
</blockquote>

<hr>
</dd>
</dl>

<blockquote>
  <p><strong>Tip:</strong> Check out the <a href="#publish-a-document"><i class="icon-upload"></i> Publish a document</a> section for a description of the different output formats.</p>
</blockquote>

<hr>

<p>You can publish your document by opening the <i class="icon-upload"></i> <strong>Publish</strong> sub-menu and by choosing a website. In the dialog box, you can choose the publication format:</p>

<ul>
<li>Markdown, to publish the Markdown text on a website that can interpret it (<strong>GitHub</strong> for instance),</li>
<li>HTML, to publish the document converted into HTML (on a blog for example),</li>
<li>Template, to have a full control of the output.</li>
</ul>

<blockquote>
  <p><strong>Note:</strong> The default template is a simple webpage wrapping your document in HTML format. You can customize it in the <strong>Advanced</strong> tab of the <i class="icon-cog"></i> <strong>Settings</strong> dialog.</p>
</blockquote>



<h3 id="tables">Tables</h3>

<p><strong>Markdown Extra</strong> has a special syntax for tables:</p>

<table>
<thead>
<tr>
  <th>Item</th>
  <th>Value</th>
</tr>
</thead>
<tbody><tr>
  <td>Computer</td>
  <td>$1600</td>
</tr>
<tr>
  <td>Phone</td>
  <td>$12</td>
</tr>
<tr>
  <td>Pipe</td>
  <td>$1</td>
</tr>
</tbody></table>


<p>You can specify column alignment with one or two colons:</p>

<table>
<thead>
<tr>
  <th align="left">Item</th>
  <th align="right">Value</th>
  <th align="center">Qty</th>
</tr>
</thead>
<tbody><tr>
  <td align="left">Computer</td>
  <td align="right">$1600</td>
  <td align="center">5</td>
</tr>
<tr>
  <td align="left">Phone</td>
  <td align="right">$12</td>
  <td align="center">12</td>
</tr>
<tr>
  <td align="left">Pipe</td>
  <td align="right">$1</td>
  <td align="center">234</td>
</tr>
</tbody></table>




<h3 id="definition-lists">Definition Lists</h3>

<p><strong>Markdown Extra</strong> has a special syntax for definition lists too:</p>

<dl>
<dt>Term 1</dt>
<dt>Term 2</dt>
<dd>Definition A</dd>

<dd>Definition B</dd>

<dt>Term 3</dt>
<dd>
<p>Definition C</p>
</dd>

<dd>
<p>Definition D</p>

<blockquote>
  <p>part of definition D</p>
</blockquote>
</dd>
</dl>



<h3 id="fenced-code-blocks">Fenced code blocks</h3>

<p>GitHub’s fenced code blocks are also supported with <strong>Highlight.js</strong> syntax highlighting:</p>



<pre class="prettyprint"><code class=" hljs cs"><span class="hljs-comment">// Foo</span>
<span class="hljs-keyword">var</span> bar = <span class="hljs-number">0</span>;</code></pre>

<blockquote>
  <p><strong>Tip:</strong> To use <strong>Prettify</strong> instead of <strong>Highlight.js</strong>, just configure the <strong>Markdown Extra</strong> extension in the <i class="icon-cog"></i> <strong>Settings</strong> dialog.</p>
  
  <p><strong>Note:</strong> You can find more information:</p>
  
  <ul>
  <li>about <strong>Prettify</strong> syntax highlighting <a href="https://code.google.com/p/google-code-prettify/">here</a>,</li>
  <li>about <strong>Highlight.js</strong> syntax highlighting <a href="http://highlightjs.org/">here</a>.</li>
  </ul>
</blockquote>



<h3 id="footnotes">Footnotes</h3>

<p>You can create footnotes like this<a href="#fn:footnote" id="fnref:footnote" title="See footnote" class="footnote">1</a>.</p>



<h3 id="smartypants">SmartyPants</h3>

<p>SmartyPants converts ASCII punctuation characters into “smart” typographic punctuation HTML entities. For example:</p>

<table>
<thead>
<tr>
  <th></th>
  <th>ASCII</th>
  <th>HTML</th>
</tr>
</thead>
<tbody><tr>
  <td>Single backticks</td>
  <td><code>'Isn't this fun?'</code></td>
  <td>‘Isn’t this fun?’</td>
</tr>
<tr>
  <td>Quotes</td>
  <td><code>"Isn't this fun?"</code></td>
  <td>“Isn’t this fun?”</td>
</tr>
<tr>
  <td>Dashes</td>
  <td><code>-- is en-dash, --- is em-dash</code></td>
  <td>– is en-dash, — is em-dash</td>
</tr>
</tbody></table>




<h3 id="table-of-contents">Table of contents</h3>

<p>You can insert a table of contents using the marker <code>[TOC]</code>:</p>

<p><div class="toc">
<ul>
<li><a href="#sperec">SpeRec</a><ul>
<li><a href="#isinma">Isinma</a><ul>
<li><ul>
<li><a href="#create-a-document"> Create a document</a></li>
</ul>
</li>
</ul>
</li>
<li><a href="#veri-hazirlama">Veri Hazirlama</a><ul>
<li><ul>
<li><a href="#switch-to-another-document"> Switch to another document</a></li>
</ul>
</li>
<li><a href="#tables">Tables</a></li>
<li><a href="#definition-lists">Definition Lists</a></li>
<li><a href="#fenced-code-blocks">Fenced code blocks</a></li>
<li><a href="#footnotes">Footnotes</a></li>
<li><a href="#smartypants">SmartyPants</a></li>
<li><a href="#table-of-contents">Table of contents</a></li>
<li><a href="#uml-diagrams">UML diagrams</a></li>
</ul>
</li>
</ul>
</li>
</ul>
</div>
</p>



<h3 id="uml-diagrams">UML diagrams</h3>

<p>You can also render sequence diagrams like this:</p>



<div class="sequence-diagram"><svg height="265" version="1.1" width="375.3203125" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" style="overflow: hidden; position: relative; top: -0.671875px;"><desc style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">Created with Raphaël 2.1.2</desc><defs style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"><path stroke-linecap="round" d="M5,0 0,2.5 5,5z" id="raphael-marker-block" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></path><marker id="raphael-marker-endblock55-obj63" markerHeight="5" markerWidth="5" orient="auto" refX="2.5" refY="2.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#raphael-marker-block" transform="rotate(180 2.5 2.5) scale(1,1)" stroke-width="1.0000" fill="#000" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></use></marker><marker id="raphael-marker-endblock55-obj69" markerHeight="5" markerWidth="5" orient="auto" refX="2.5" refY="2.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#raphael-marker-block" transform="rotate(180 2.5 2.5) scale(1,1)" stroke-width="1.0000" fill="#000" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></use></marker></defs><rect x="10" y="20" width="49.625" height="39" rx="0" ry="0" fill="none" stroke="#000000" stroke-width="2" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><rect x="19.984375" y="30" width="29.625" height="19" rx="0" ry="0" fill="#ffffff" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><text x="34.8125" y="39.5" text-anchor="middle" font-family="Andale Mono, monospace" font-size="16px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: middle; font-family: &quot;Andale Mono&quot;, monospace; font-size: 16px;"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">Alice</tspan></text><rect x="10" y="206" width="49.625" height="39" rx="0" ry="0" fill="none" stroke="#000000" stroke-width="2" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><rect x="19.984375" y="216" width="29.625" height="19" rx="0" ry="0" fill="#ffffff" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><text x="34.8125" y="225.5" text-anchor="middle" font-family="Andale Mono, monospace" font-size="16px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: middle; font-family: &quot;Andale Mono&quot;, monospace; font-size: 16px;"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">Alice</tspan></text><path fill="none" stroke="#000000" d="M34.8125,59L34.8125,206" stroke-width="2" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></path><rect x="182.9296875" y="20" width="45.265625" height="39" rx="0" ry="0" fill="none" stroke="#000000" stroke-width="2" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><rect x="192.921875" y="30" width="25.265625" height="19" rx="0" ry="0" fill="#ffffff" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><text x="205.5625" y="39.5" text-anchor="middle" font-family="Andale Mono, monospace" font-size="16px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: middle; font-family: &quot;Andale Mono&quot;, monospace; font-size: 16px;"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">Bob</tspan></text><rect x="182.9296875" y="206" width="45.265625" height="39" rx="0" ry="0" fill="none" stroke="#000000" stroke-width="2" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><rect x="192.921875" y="216" width="25.265625" height="19" rx="0" ry="0" fill="#ffffff" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><text x="205.5625" y="225.5" text-anchor="middle" font-family="Andale Mono, monospace" font-size="16px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: middle; font-family: &quot;Andale Mono&quot;, monospace; font-size: 16px;"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">Bob</tspan></text><path fill="none" stroke="#000000" d="M205.5625,59L205.5625,206" stroke-width="2" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></path><rect x="44.796875" y="74.5" width="150.75" height="19" rx="0" ry="0" fill="#ffffff" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><text x="120.1875" y="84" text-anchor="middle" font-family="Andale Mono, monospace" font-size="16px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: middle; font-family: &quot;Andale Mono&quot;, monospace; font-size: 16px;"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">Hello Bob, how are you?</tspan></text><path fill="none" stroke="#000000" d="M34.8125,98C34.8125,98,172.41555504500866,98,200.56774257618963,98" stroke-width="2" marker-end="url(#raphael-marker-endblock55-obj63)" stroke-dasharray="0" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></path><rect x="225.5625" y="118" width="77.125" height="29" rx="0" ry="0" fill="none" stroke="#000000" stroke-width="2" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><rect x="230.546875" y="123" width="67.125" height="19" rx="0" ry="0" fill="#ffffff" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><text x="264.125" y="132.5" text-anchor="middle" font-family="Andale Mono, monospace" font-size="16px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: middle; font-family: &quot;Andale Mono&quot;, monospace; font-size: 16px;"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">Bob thinks</tspan></text><rect x="64.0625" y="162.5" width="112.234375" height="19" rx="0" ry="0" fill="#ffffff" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></rect><text x="120.1875" y="172" text-anchor="middle" font-family="Andale Mono, monospace" font-size="16px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: middle; font-family: &quot;Andale Mono&quot;, monospace; font-size: 16px;"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">I am good thanks!</tspan></text><path fill="none" stroke="#000000" d="M205.5625,186C205.5625,186,67.95944495499134,186,39.80725742381037,186" stroke-width="2" marker-end="url(#raphael-marker-endblock55-obj69)" stroke-dasharray="6,2" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></path></svg></div>

<p>And flow charts like this:</p>



<div class="flow-chart"><svg height="402.1640625" version="1.1" width="171.53125" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" style="overflow: hidden; position: relative; top: -0.265625px;"><desc style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">Created with Raphaël 2.1.2</desc><defs style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"><marker id="raphael-marker-endblock33-obj78" markerHeight="3" markerWidth="3" orient="auto" refX="1.5" refY="1.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#raphael-marker-block" transform="rotate(180 1.5 1.5) scale(0.6,0.6)" stroke-width="1.6667" fill="black" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></use></marker><marker id="raphael-marker-endblock33-obj79" markerHeight="3" markerWidth="3" orient="auto" refX="1.5" refY="1.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#raphael-marker-block" transform="rotate(180 1.5 1.5) scale(0.6,0.6)" stroke-width="1.6667" fill="black" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></use></marker><marker id="raphael-marker-endblock33-obj80" markerHeight="3" markerWidth="3" orient="auto" refX="1.5" refY="1.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#raphael-marker-block" transform="rotate(180 1.5 1.5) scale(0.6,0.6)" stroke-width="1.6667" fill="black" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></use></marker><marker id="raphael-marker-endblock33-obj82" markerHeight="3" markerWidth="3" orient="auto" refX="1.5" refY="1.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#raphael-marker-block" transform="rotate(180 1.5 1.5) scale(0.6,0.6)" stroke-width="1.6667" fill="black" stroke="none" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></use></marker></defs><rect x="0" y="0" width="50.75" height="39" rx="20" ry="20" fill="#ffffff" stroke="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);" stroke-width="2" class="flowchart" id="st" transform="matrix(1,0,0,1,48.8906,19.6328)"></rect><text x="10" y="19.5" text-anchor="start" font-family="sans-serif" font-size="14px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: start; font-family: sans-serif; font-size: 14px; font-weight: normal;" id="stt" class="flowchartt" font-weight="normal" transform="matrix(1,0,0,1,48.8906,19.6328)"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">Start</tspan></text><rect x="0" y="0" width="104.265625" height="39" rx="0" ry="0" fill="#ffffff" stroke="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);" stroke-width="2" class="flowchart" id="op" transform="matrix(1,0,0,1,22.1328,128.2656)"></rect><text x="10" y="19.5" text-anchor="start" font-family="sans-serif" font-size="14px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: start; font-family: sans-serif; font-size: 14px; font-weight: normal;" id="opt" class="flowchartt" font-weight="normal" transform="matrix(1,0,0,1,22.1328,128.2656)"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">My Operation</tspan></text><path fill="#ffffff" stroke="#000000" d="M35.1328125,17.56640625L0,35.1328125L70.265625,70.265625L140.53125,35.1328125L70.265625,0L0,35.1328125" stroke-width="2" font-family="sans-serif" font-weight="normal" id="cond" class="flowchart" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); font-family: sans-serif; font-weight: normal;" transform="matrix(1,0,0,1,4,221.2656)"></path><text x="40.1328125" y="35.1328125" text-anchor="start" font-family="sans-serif" font-size="14px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: start; font-family: sans-serif; font-size: 14px; font-weight: normal;" id="condt" class="flowchartt" font-weight="normal" transform="matrix(1,0,0,1,4,221.2656)"><tspan dy="5.5078125" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">Yes or No?</tspan><tspan dy="18" x="40.1328125" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></tspan></text><rect x="0" y="0" width="44.421875" height="39" rx="20" ry="20" fill="#ffffff" stroke="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);" stroke-width="2" class="flowchart" id="e" transform="matrix(1,0,0,1,52.0547,361.1641)"></rect><text x="10" y="19.5" text-anchor="start" font-family="sans-serif" font-size="14px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: start; font-family: sans-serif; font-size: 14px; font-weight: normal;" id="et" class="flowchartt" font-weight="normal" transform="matrix(1,0,0,1,52.0547,361.1641)"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">End</tspan></text><path fill="none" stroke="#000000" d="M74.265625,58.6328125C74.265625,58.6328125,74.265625,111.9454345703125,74.265625,125.27359008789062" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj78)" font-family="sans-serif" font-weight="normal" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); font-family: sans-serif; font-weight: normal;"></path><path fill="none" stroke="#000000" d="M74.265625,167.265625C74.265625,167.265625,74.265625,206.91972494125366,74.265625,218.26606408460066" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj79)" font-family="sans-serif" font-weight="normal" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); font-family: sans-serif; font-weight: normal;"></path><path fill="none" stroke="#000000" d="M74.265625,291.53125C74.265625,291.53125,74.265625,344.8438720703125,74.265625,358.1720275878906" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj80)" font-family="sans-serif" font-weight="normal" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); font-family: sans-serif; font-weight: normal;"></path><text x="79.265625" y="301.53125" text-anchor="start" font-family="sans-serif" font-size="14px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: start; font-family: sans-serif; font-size: 14px; font-weight: normal;" font-weight="normal"><tspan dy="5.5" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">yes</tspan></text><path fill="none" stroke="#000000" d="M144.53125,256.3984375C144.53125,256.3984375,169.53125,256.3984375,169.53125,256.3984375C169.53125,256.3984375,169.53125,103.265625,169.53125,103.265625C169.53125,103.265625,74.265625,103.265625,74.265625,103.265625C74.265625,103.265625,74.265625,118.63906955718994,74.265625,125.27487277425826" stroke-width="2" marker-end="url(#raphael-marker-endblock33-obj82)" font-family="sans-serif" font-weight="normal" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); font-family: sans-serif; font-weight: normal;"></path><text x="149.53125" y="246.3984375" text-anchor="start" font-family="sans-serif" font-size="14px" stroke="none" fill="#000000" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0); text-anchor: start; font-family: sans-serif; font-size: 14px; font-weight: normal;" font-weight="normal"><tspan dy="5.5078125" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);">no</tspan></text></svg></div>

<div class="footnotes"><hr><ol><li id="fn:footnote">Here is the <em>text</em> of the <strong>footnote</strong>. <a href="#fnref:footnote" title="Return to article" class="reversefootnote">↩</a></li></ol></div>