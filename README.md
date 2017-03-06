SpeRec
===================
**KALDI** ve **Java** kullanarak, adim adim konusmadan metine (text-to-speech) programi gelistirme rehberi! 

---------


Isinma
-------------

İlk olarak gerekli üçüncü parti yazılımları yükleyip çalışma ortamımızı kuracağız. Sonrasında ihtiyacımız olan verilerden bahsedip (ki burada **herkes kendi ses kaydını kullanacak**), ilk modelimizi oluşturacağız. Son aşamada ise -*Java sadece burada devreye giriyor*- kaydettiğimiz sesleri ekrana bastıracağız. 

> **On Kosullar:**

> - Linux İşletim Sistemi

> - Temel Linux Shell kullanımı

> - Temel Java - Swing kodlama

----------

Gerekli Yazilimlari Yükleme
#### <i class="icon-file"></i> Create a document

Bu rehber, kullanmakta olduğnuz linux dagitimin'da aşağıdaki paketlerin yüklü olduğunu varsayar. Eğer emin değilseniz aşağıdaki scripti kuruluma başlamadan önce çalıştırın: 
sudo apt-get install atlas autoconf automake git libtool svn wget zlib
 
KALDI'yı indirme: 
git clone https://github.com/kaldi-asr/kaldi

Bu yüklemenin ardından dosyayı indirdiğiniz konumdaki (aksini belitrmediyseniz /home/{kullanıcı adınız} klasörü) "kaldı" klasörünün içeriği:


----------

<i class="icon-folder-open"></i>/egs

:   > Dokumantasyonlari ile birlikte 44 farklı örnek proje. Malesef birkaçı dışında ses dosyalarını indirmek ücretli.

<i class="icon-folder-open"></i>/misc

:   > Çeşitli kaynak ve pdf dosyalarını içeren, daha derine KALDI öğrenmek isteyenler için faydalı kaynaklar içeren bir klasör. Ayrıca resmi logo ve HTK'dan KALDI'ye çevirme yapan scriptler içerir. (Kabul, hiç girmedim!)

<i class="icon-folder-open"></i>/src

:   > KALDI kaynak kodu

<i class="icon-folder-open"></i>/tools

:   > ATLAS, SRILM gibi KALDI'nın kullandığı üçüncü parti yazılımlar. 

<i class="icon-folder-open"></i>/windows

:   > KALDI'yı Windows'ta kulanmak için gerekli araçlar. (Malesef, bu rehberde geçen son Windows kelimesi) 
	


----------

İndirme işleminden sonra gelelim kuruluma. Gerekli adımlar INSTALL dosyasında yazıyor. Tek adımda yapmak için

          cd kaldi/tools/; make; cd ../src; ./configure; make

scripti yeterli. Bu adımda build yapıldığından biraz vakit alabilir, kahvenizi yenileyin!















# SpeRec
Akustik Model nedir?
Language Model nedir?
SRILM Tool nedir?
ARPA formati nedir?
n-gram?
perplexity
word error rate
  ------------------

# KALDI kurulumu
svn(subversion)'in kurulu oldugundan emin olun.
Terminalden *BURAYI TERMINAL GORSELLI YAP*: 
svn co https://kaldi.svn.sourceforge.net/svnroot/kaldi/trunk kaldi-trunk
  veya git ile:
git clone https://github.com/kaldi-asr/kaldi.git kaldi-trunk --origin golden
cd kaldi-trunk/tools
extras/check_dependencies.sh
./install.sh
cd ../src
./configure
make -j 8 ## -j parametresi processi paralel olarak boler. verilen deger makinanin cekirdek sayisindan buyuk olmamalidir.

tools icinde:
  -OpenFST: weighted finite state transducer library
  -Atlas : Standart linear algebra libraries
  
  ------------------
# Data Hazirlama
 
 waw.scp:
   <ID> <path to .wav file> 
   00001 /home/user/00001.wav
 text:
   <ID> <Transciption>
   00001 fatura bilgilerim
 spk2utt & utt2spk:
   <ID> <ID> 
   00001 00001
 corpus:
   <s> <TRANSCRIPT> </s> 
   <s> fatura bilgilerim </s>
 lexicon:
    <Word> <Phonemes>
    güncel g ü n c e l
 
   ------------------
## run.sh

source ./path.sh
. ./cmd.sh
KALDI_ROOT=/home/USER/kaldi-trunk/ # USER'i kullanici adiniz ile degistirin.
train_cmd=run.pl
decode_cmd=run.pl
featdir=data/mfcc
mkdir -p $featdir
train_nj=8 # paralel calisacak is sayisi


# Feature extraction
steps/make_mfcc.sh --cmd "$train_cmd" --nj $train_nj data/train exp/make_mfcc/train $featdir

#test datasi icin ekledim. Create MFCC features for test
steps/make_mfcc.sh --nj $train_nj --cmd "$train_cmd" data/test exp/make_mfcc/test $featdir
  
# Feature extraction
#steps/compute_cmvn_stats.sh data/train exp/make_plp/train $featdir 
steps/compute_cmvn_stats.sh data/train exp/make_mfcc/train $featdir 
#test datasi icin ekledim. Making cmvn.scp(CMVN stats) files:
steps/compute_cmvn_stats.sh data/test exp/make_mfcc/test $featdir

#Yine tutorial'dan data duzenleme:
#utils/validate_data_dir.sh data/train
#utils/fix_data_dir.sh data/train


# Data preparation
utils/prepare_lang.sh --position-dependent-phones false data/local/dict "oov" data/local/lang data/lang || exit 1;

#utils/subset_data_dir.sh data/train 1000000 data/train.1M || exit 1;

## "===== LANGUAGE MODEL CREATION ====="
echo "===== MAKING lm.arpa ====="
echo

loc=`which ngram-count`;
if [ -z $loc ]; then
     if uname -a | grep 64 >/dev/null; then
        sdir=$KALDI_ROOT/tools/srilm/bin/i686-m64
    else
            sdir=$KALDI_ROOT/tools/srilm/bin/i686
      fi
      if [ -f $sdir/ngram-count ]; then
            echo "Using SRILM language modelling tool from $sdir"
            export PATH=$PATH:$sdir
      else
            echo "SRILM toolkit is probably not installed.
              Instructions: tools/install_srilm.sh"
            exit 1
      fi
fi
local=data/local
mkdir $local/tmp
ngram-count -order 3 -write-vocab $local/tmp/vocab-full.txt -wbdiscount -text $local/corpus.txt -lm $local/tmp/lm.arpa

######


## "===== MAKING G.fst ====="
lang=data/lang
cat $local/tmp/lm.arpa | arpa2fst - | fstprint | utils/eps2disambig.pl | utils/s2eps.pl | fstcompile --isymbols=$lang/words.txt --osymbols=$lang/words.txt --keep_isymbols=false --keep_osymbols=false | fstrmepsilon | fstarcsort --sort_type=ilabel > $lang/G.fst

# Mono training
steps/train_mono.sh --nj $train_nj --cmd "$train_cmd" data/train data/lang exp/mono  || exit 1;

# "===== MONO DECODING ====="

utils/mkgraph.sh --mono data/lang exp/mono exp/mono/graph || exit 1;
steps/decode.sh --config conf/decode.config --nj $train_nj --cmd "$decode_cmd" exp/mono/graph data/test exp/mono/decode

# Get alignments from monophone system.
steps/align_si.sh --use-graphs false --nj $train_nj --cmd "$train_cmd" \
data/train data/lang exp/mono exp/mono_ali || exit 1;

# train tri1 [first triphone pass]
steps/train_deltas.sh --cmd "$train_cmd" \
200 1000 data/train data/lang exp/mono_ali exp/tri1 || exit 1;

# align tri1
steps/align_si.sh --nj $train_nj --cmd "$train_cmd" \
--use-graphs true data/train data/lang exp/tri1 exp/tri1_ali || exit 1;

# train tri2 [delta+delta-deltas]
steps/train_deltas.sh --cmd "$train_cmd" 200 1000 \
data/train data/lang exp/tri1_ali exp/tri2 || exit 1;

# align tri2
steps/align_si.sh --nj $train_nj --cmd "$train_cmd" \
--use-graphs true data/train data/lang exp/tri2 exp/tri2_ali || exit 1;

# train tri3 [delta+delta-deltas]
steps/train_deltas.sh --cmd "$train_cmd" 200 1000 \
data/train data/lang exp/tri2_ali exp/tri3 || exit 1;

# align tri3
steps/align_si.sh --nj $train_nj --cmd "$train_cmd" \
--use-graphs true data/train data/lang exp/tri3 exp/tri3_ali || exit 


# train triLDA [LDA+MLLT]
steps/train_lda_mllt.sh --cmd "$train_cmd" \
  --splice-opts "--left-context=3 --right-context=3" \
 200 1000 data/train data/lang exp/tri3_ali exp/triLDA || exit 1;

# tri2b=triLDA 
# tri2b_mmi=triMMI

# Align all data with LDA+MLLT system (tri2b)
steps/align_si.sh --nj $train_nj --cmd "$train_cmd" \
   data/train data/lang exp/triLDA exp/triLDA_ali || exit 1;

#  Do MMI on top of LDA+MLLT.
steps/make_denlats.sh --nj $train_nj --cmd "$train_cmd" \
  data/train data/lang exp/triLDA exp/triLDA_denlats || exit 1;

steps/train_mmi.sh data/train data/lang exp/triLDA_ali exp/triLDA_denlats exp/triMMI || exit 1;

steps/align_si.sh --nj $train_nj --cmd "$train_cmd" \
--use-graphs false data/train data/lang exp/triMMI exp/triMMI_ali || exit 



