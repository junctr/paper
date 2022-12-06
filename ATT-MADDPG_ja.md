チームメイトの動的な共同方針のモデル化 注意マルチエージェントDDPG

Abstract

協調型マルチエージェントシステムにおけるチームメイトの政策のモデリングと活用は、強化学習（RL）コミュニティにとって長年の関心事であると同時に大きな挑戦でもあった。その理由は、エージェントがチームメイトの政策を知っていれば、それに応じて自分の政策を調整し、適切な協力関係を構築できることにある。一方、課題は、エージェントの政策が同時学習により絶えず変化するため、チームメイトの動的な政策を正確にモデル化することが困難であることである。本論文では、この課題を解決するために、ATTention Multi-Agent Deep Deterministic Policy Gradient (ATT-MADDPG)を発表する。ATT-MADDPGは、シングルエージェントのアクタークリティックRL手法であるDDPGを、2つの特別な設計で拡張したものである。まず、チームメイトの政策をモデル化するために、エージェントはチームメイトの観測と行動にアクセスする必要があります。ATT-MADDPGでは、このような情報を収集するために、中央集権的な批判者を採用する。第二に、収集した情報を用いてチームメイトの政策を効果的にモデル化するために、ATT-MADDPGは注目機構によって集中型批判者を強化する。このアテンション機構は、チームメイトの動的な共同政策を明示的にモデル化するための特別な構造を導入し、収集した情報を効率的に処理できるようにするものである。我々は、ATT-MADDPGをベンチマークタスクと実世界のパケットルーチングタスクの両方で評価した。実験の結果、最先端のRLベース手法やルールベース手法を大きく上回るだけでなく、スケーラビリティやロバスト性の面でもより良い性能を達成することがわかった。

1 introduction

ネットワークパケットルーティング(Vicisano et al., 1998; Tao et al., 2001)、自律交差点管理(Dresner and Stone, 2008)、ポーカーゲーム(Billings et al., 1998)など、複数のエージェントが関わる実世界のタスクは多数存在する。このようなマルチエージェントタスクに対して、強化学習(RL) (Sutton and Barto, 1998)を適用する試みは、過去数十年にわたり続けられてきた。なぜなら、これらのタスクを学習ベースの手法で解くことは、人工知能システムの構築に不可欠なステップであるからである。しかしながら、エージェントの部分的な観測可能性、エージェント間の協調と競争、エージェント数の変化など、多くの課題があるため、未解決のままである。

本論文では、協調的な分散型マルチエージェントRL設定に注目する。協調的な設定では、エージェントは共有されたゴールを達成するために協調的な行動をとる必要がある。分散環境では、エージェントは部分的に観測可能な異なる地域に配置される。代表的なタスクはパケットルーティングであり、ルータは自律エージェントとして扱われ、目標はできるだけ少ないリソースでパケットを送信することである。

このように単純化された設定であっても、エージェントのモデリング問題が複雑であるため、このようなタスクを扱うことは困難である（Albrecht and Stone, 2018）。具体的には、エージェントがチームメイトのポリシーに関するモデルを保持していれば、それに応じて自身のポリシーを調整し、適切な協力を実現することができる。しかし、エージェントは同じ環境で同時並行的に学習しているため、その方針は連続的に変化している。このように動的に変化する政策を正確にモデル化することは非常に困難である。仮になんとかできたとしても、どうせ簡単に時代遅れになってしまう。

実際、この優れたサーベイ（Albrecht and Stone, 2018）にまとめられているように、チームメイトのポリシーをモデル化して利用することは、RLコミュニティにとって長い間の関心事でした。とはいえ、ほとんどの手法はゲーム理論（Ganzfried and Sandholm, 2011）または単純なグリッドワールドの設定で導入されており、通常、各チームメイトの政策を個別にモデル化しています。これらの手法は、ネットワークパケットルーティングのような実世界のアプリケーションに拡張することが困難である。

近年、大規模なタスクを対象とした深層強化学習（Deep Reinforcement Learning: DRL）が検討されている。状態空間と行動空間が広いタスクで汎化するために、DRLに基づく方法は、類似した状態に対して類似した行動を生成する関数近似器としてディープニューラルネットワークを採用する。しかし、既存のDRLベースのエージェントモデリング手法（He et al., 2016; Foerster et al., 2018; Raileanu et al., 2018; Yang et al., 2018; Hong et al., 2018）は、ほとんどがディープQネットワーク（DQN）（Mnih et al., 2015）の改善に焦点を当てており、通常、集中型ポリシーを学習しています。分散システムにおいて中央集権的なポリシーを適用するためには、エージェントは実行中に情報を交換する必要があり、これは多くの場合、コストがかかりすぎ、あるいは実現不可能である（Roth et al., 2005; Zhang and Lesser, 2013; Chen et al.） さらに、DQNに基づく方法は、離散的な行動空間を持つタスクに対処することを目標としている。他のDQNに基づく手法（Sunehag et al., 2017; Rashid et al., 2018）やアクタークリティックRLアルゴリズムに基づく研究（Foerster et al., 2017; Lowe et al., 2017; Chu and Ye, 2017）は分散型ポリシーを生成できるが、他のエージェントに対するモデルを明確に構築していない。その代わり、彼らは複数のエージェント間の信用割り当てなど、他のトピックを調査している。

本論文では、複雑なエージェントのモデリング問題に対処するために、ATTention Multi-Agent Deep Deterministic Policy Gradient (ATTMADDPG) を紹介する。先行研究とは対照的に、ATTMADDPGはチームメイトの動的な共同方針を適応的に明示的にモデル化し、連続的な行動空間を持つ大規模分散タスクを扱う分散型方針の学習のために設計されたものである。

具体的には、ATT-MADDPGは、シングルエージェントのアクター・クリティックRLアルゴリズムであるDDPG (Lillicrap et al., 2015) を、2つの特別な設計で拡張しています。まず、エージェントモデリングを行うために必要なステップとして、エージェントはチームメイトの観測と行動へのアクセスを得る必要があります。ATT-MADDPGはこれらの情報を収集するために中央集権的な批判者を採用する。第二に、収集した情報を効果的に処理してチームメイトの政策をモデル化するために、ATT-MADDPGではさらに注目機構を集中型クリティストに組み込んでいます。この注意機構は、チームメイトの動的な共同方針を適応的に明示的にモデル化するための特別な構造を導入しています。チームメイトが政策を変更すると、関連する注意の重みが適応的に変更され、エージェントは迅速に政策を調整することができる。その結果、すべてのエージェントが効率的に協力することになる。また、DDPGは連続的な行動空間のタスクを対象としているので、ATT-MADDPGはそのようなタスクに自然に対応することができる。さらに、DDPGのアクター部分を変更しないため、政策が分散化され、アクターは自身の観測履歴に基づき行動を生成することができる。

我々は、ATT-MADDPGを実世界のパケットルーチングタスク、および、ベンチマークの協調ナビゲーションと捕食者捕食タスクで評価した。全てのタスクにおいて、ATT-MADDPGは最先端のRLベースの手法とルールベースの手法の両方よりも多くの報酬を得ることができる。また、ATT-MADDPGはより良いスケーラビリティとロバスト性を達成することが実験で示された。さらに、パケットルーチングタスクでは注意のメカニズムについて、協調航法タスクではエージェントの政策間の協調について実験を行い、いくつかの知見を得ることができた。

我々の主な貢献は以下のようにまとめられる。
- 多くのエージェントモデリング手法とは対照的に、ATT-MADDPGは、連続的な行動を伴う分散タスクを処理するために、各エージェントに対して分散化されたポリシーを学習させる。
- 提案する注意メカニズムは、チームメイトの動的な共同方針を適応的に明示的にモデル化するための特別な構造を導入しています。我々の知る限り、この新しい方法でエージェントモデリングを行ったのは我々が最初である。
- 我々は、ATT-MADDPGを実世界のタスクとベンチマークタスクの両方で実証的にテストし、報酬、スケーラビリティ、ロバスト性の面で良好な性能を達成することを示す。

2 background

DEC-POMDP。我々はDEC-POMDP (Bernstein et al., 2002)として定式化できるマルチエージェントの設定を考えている。ここで、N はエージェントの数、S は状態 s の集合、A~ = [A1, ..., AN ] は共同行動 ~a の集合、Ai はエージェント i がとることのできる局所行動 ai の集合、T（s 0 |s, ~a） ... S × A~ × S → [0, 1] は状態遷移関数、Ai はエージェント i がとることのできる局所行動 ai の 集合を表す。S × A~ × S → [0, 1] は状態遷移関数、R~ = [R1, ..., RN ] : S×A~ → R N は共同報酬関数、O~ = [O1, ..., ON ] は観測関数 Z : S × A~ → O~ で制御される共同観測 ~o の集合、γ ∈ [0, 1] は割引因子である。　

与えられた状態 s において、各エージェントは自身の観測（履歴） oi に基づいて行動 ai をとり、その結果、新しい状態 s 0 と報酬 ri 2 が得られる。エージェントは政策 πi を学習しようとする．ここで、Gi は Gi = PH t=0 γ t r t i と定義される割引リターン、H は時間地平である。また、環境は共同完全観測可能（Bernstein et al.2002）、すなわち、s , ~o = hoi , ~o-ii （~o-i はエージェント i のチームメイトの共同観測（履歴））と仮定する。

RL (Sutton and Barto, 1998) は一般にN = 1の特殊なDEC-POMDP問題を解くために用いられる。実際には，Q 値関数 Qπ (s, a) を (((1))) と定義し，π ＊ = arg maxπ Qπ (s, a) で最適政策を導出する．　

政策勾配法（Sutton et al, 2000) は，任意の政策 π の近似であるパラメタ化政策 πθ = π(a|s; θ) を直接学習する．目的J(θ) = Es∼pπ,a∼πθ [G]を最大化するために、pπを安定状態分布とし、∇θJ(θ) = Es∼pπ,a∼πθ [∇θ log π(a|s; θ)Qπ (s, a)] の方向へパラメータθを調整する。Qπ (s, a)を近似するためにディープニューラルネットワークQ(s, a; w)を用いることができ、その結果、アクター批判アルゴリズム(Konda and Tsitsiklis, 2003; Grondman et al., 2012)を実現することができる。パラメータ化されたアクターπ(a|s; θ)とクリティックQ(s, a; w)の両方が学習時に用いられ、実行時にはアクターπ(a|s; θ)のみが必要とされる。このメリットは本手法において分散化された政策を学習する際に利用される。　

決定論的政策勾配 (DPG) (Silver et al., 2014) は、アクターが決定論的政策 µθ : S → A を採用し、行動空間 A が連続的である特殊なアクター・クリティック・アルゴリズムである。Deep DPG (DDPG) (Lillicrap et al., 2015) は、ディープニューラルネットワークを用いて、μθ(s) と Q(s, a; w) を近似するものである。　DDPGはオフポリシー法であり、ターゲットネットワークと経験リプレイを適用して学習を安定化させ、データ効率を向上させる。具体的には、Critic と Actor は以下の式に基づいて更新される： 
(((2))) (((3))) (((4)))

D は最近の経験タプル (s, a, r, s0 ) を含む再生バッファ； Q(s, a; w -) および µθ- (s) はパラメータ w - および θ - をコピーして周期的に更新するターゲットネットワークである。　

注意のメカニズム 図1に示すように、Soft Attention (Xu et al., 2015) (Global Attention (Luong et al., 2015) と呼ばれることもある) が最もポピュラーである。入力は複数のソースベクトル[S1, S2, ..., Sk, ..., SK]と一つのターゲットベクトルTであり、モデルはより重要なSkに適応的に出席することができ、重要度はユーザ定義の関数f（T, Sk）によって測定され；Skに含まれる重要情報は以下のように正規化重要度スコアwkに従って適応的に文脈ベクトルCに符号化され得る。　
(((5)))　

また、注目重みベクトルW 、［w1、w2、...、wk、...、wK］は、PK k=1 wk ≡ 1なので、確率分布と見ることもできる。この確率分布を適応的に生成する工夫が本手法に適用される。　

3 attention multi-agent ddpg

詳細を掘り下げる前に、本論文で使用する主要な変数を表1に列挙する。なお、~π-i, ~π-i(~a-i|s), ~π-i(A~-i|s) の違いにご注目ください。

3.1 the overall approach

提案するATT-MADDPGは、行為者批判型RLアルゴリズムを集中批判者と注意メカニズムで拡張したものである。我々の方法をより理解しやすくするために、注意メカニズムを考慮せずに全体のアプローチを示す。注目機構については次節で紹介する。

具体的には、図 2 からわかるように、集中批判者 Qi （すなわち、エージェント i に関係する Q 値関数）はすべてのエージェントの観測と行動にアクセスすることができるが、独立アクター πi は自分自身の観測 oi にしかアクセスすることができない。従って、ATT-MADDPGは学習時に以下のように動作する。

ステップ1：行為者πiは、環境と相互作用するために、自身の観測値oiに基づき行動aiを生成する。

ステップ2：中央の評定者は、全エージェントの観測値と行動に基づいてQ値Qiを推定する。

ステップ 3: 環境からのフィードバック報酬を受け取った後、式 10、11、12 に基づく逆伝播法 (BP) を用いて、行為者と批評家を共同で学習させる。

一般に、他のエージェントの観測値 ~o-i と行動 ~a-i にアクセスせずに、他のエージェントの政策をモデル化する方法はない；ステップ2において、集中型批評家 Qi は ~o-i と ~a-i を集めるように設計されており、エージェントモデル化のために必要な基礎を形成している。さらに、中央集権的な評論家により、エージェントは安定した報酬信号riで学習することができるため、我々の方法は非定常問題を緩和することもできる（Weinberg and Rosenschein, 2004; Hernandez-Leal et al, 2017）3.

3.2 the attention critic

適切な協力を得るために、エージェントはチームメイトの政策をモデル化し、それに応じて自身の政策を調整することが期待される。我々は、チームメイトの動的な共同政策が適応的にモデル化されるように、一種のSoft Attentionを設計し、中央集権的な批判者に埋め込む。

我々の設計をより理解しやすくするために、行動は離散的であるという仮定に基づいて紹介する。連続的な行動への拡張はセクション 3.3 で示す。

マルチエージェントの設定において、環境は〜a によって影響されることを想起する。エージェント i の観点からは、s で行われる ai の結果は ~a-i に依存する。そこで，式1のQπ（s，a）の定義と同様に，先行研究（He et al., 2016; Banerjee and Sen, 2007）と同様にチームメイトの共同政策に対するQ値関数をQ πi|~π-i i (s, ai) と定義し，新しい目的は最適政策π ＊ i = arg maxπi Q πi|~π-i i (s, ai) を見出すことである．数学的には、Q πi|~π-i i (s, ai)は、(((6))) (((7))) によって計算できる。

式7は、Q πi|~π-i i (s, ai)を推定するために、エージェントiの評論家ネットワークは以下の能力を持つべきであることを意味する： 
(((1))) 各~a-i ∈ A~-i についてQ πi i (s, ai , ~a-i) を推定すること。
(((2)))すべてのQ πi i (s, ai , ~a-i) の期待値を計算する 5 . 

各~a-i ∈ A~-i のQ πi i (s, ai , ~a-i) を推定するために、K=|A~-i | とするK-head Moduleを設計する。図3の下部に示すように、Kヘッドモジュールは、エージェントiの評論家ネットワークのパラメータをwiとしたとき、真のQ πi i (s, ai , ~a-i) を近似するために、各~a-iに対してK個の行動条件Q値Qk i (s, ai |~a-i ; wi) を発生させる。具体的には，Qk i (s, ai |~a-i ; wi) は，ai とすべての観測値 hoi , ~o-ii = ~o , sを用いて生成される．~a-i に関する情報については，まもなく紹介する追加の隠れベクトル hi(wi) によって与えられる6 ．

また、全てのQ πi i (s, ai , ~a-i) の期待値を計算するためには、式7で示されるように、全てのQ πi i (s, ai , ~a-i) の重み ~π-i(~a-i |s)が必要である。しかし、これらの重みを近似的に求めることは困難である。一方，異なる s に対して，チームメイトは政策 ~π-i に基づいて異なる確率 ~π-i(~a-i |s) で異なる ~a-i を取ることになる．一方、エージェントは互いに適応するために同時学習を行うので、政策 ~π-i は連続的に変化する。

我々は、すべての〜π-i(〜a-i |s) ∈ ~π-i(A~-i |s) を、エージェントiの批判ネットワークのパラメータである重みベクトルWi(wi) , [W1 i (wi), ..., W K i (wi)] によって合同に近似することを提案する。つまり、各確率値〜π-i（〜a-i｜s）を個別に近似するのではなく、Wi（wi）を用いて確率分布〜π-i（A〜-i｜s）を近似するのである。良いWi（wi）は以下の条件を満たすべきである：（1）Σ K k=1Wk i（wi）≡1、Wi（wi）が確かに確率分布であること、（2）Wi（wi）が本当にチームメイトの共同方針を適応的にモデル化できるように、チームメイトの共同方針が変更されたときに適応的に変更できること、などである。

アテンション機構は本質的に確率分布を適応的に生成するのに適している（第2節参照）ので、これを利用してアテンションモジュールを設計する。図3中段に示すように、アテンションモジュールは以下のように動作する。

まず、チームメイトの全行動（〜a-i）に基づいて隠れベクトルhi(wi)が生成される。

次に、hi(wi)と全ての行動条件Q値Qk i (s, ai |~a-i ; wi)を比較し、注意重みベクトルWi(wi)を生成する。具体的には、ドットスコア関数（Luong et al., 2015）を適用して、要素Wk i (wi) ∈ Wi(wi): (((8))) を算出する。

最後に、文脈Q値Qc i (s, ai , ~a-i ; wi)をWk iとQk iの加重和として計算する : (((9))) 

まとめ：式7ではチームメイトを考慮したが、式9はWk i (wi) とQk i (s, ai |~a-i ; wi) がそれぞれ ~π-i(~a-i |s) とQ πi i (s, ai , ~a-i) を近似的に学習できるので式7を近似したものである。このように、ATT-MADDPGによって制御されるエージェントは、効率的に協力することができる。

3.3 key implementation

アテンションモジュール。文脈Q値Qc i (s, ai , ~a-i ; wi)を得た後、図3の上部に示すように、1つの出力ニューロンを持つ完全連結層を用いて、多次元Qc iをスカラー実Q値Qiに変換する必要があります。

これは、多くの研究により、Soft Attentionを実装する場合、スカラーよりも多次元ベクトルの方が有効であることが示されているからである（Xu et al.、2015）。我々のAttention Moduleにおいても、スカラーよりもベクトルの方がうまく機能することが分かっており、Qc i , Qk i , hi(wi) , Wi(wi) は全てベクトルを用いて実装されています。しかし、標準的なRLはスカラー実Q値Qiを採用しているので、Qc iをスカラー実Q値Qiに変換する必要がある。

K-headモジュール 以上の議論は、離散作用空間に限定したものである。ここで当然の疑問として、Qk i (s, ai |~a-i ; wi)を各~a-i∈A~-iに対して1つ生成すればよいのだろうか？行動空間が連続的であればどうだろうか？

実は、K = |A~-i|とする必要はない。多くの研究者が、ほとんどの場合、小さなアクションのセットだけが重要であることを示し、この結論は連続（Silver et al.、2014）と離散（Wang et al.、2015）の両方のアクション空間環境に適している。

Qk i (s, ai |~a-i ; wi)が類似の~a-iをグループ化できれば（つまり、異なるが類似の~a-iを一つのQ値ヘッドで表現できれば）、より効率的になると主張する。ディープニューラルネットワークは普遍的な関数近似器であるため(Cybenko, 1989; Hornik et al., 1989; Schaul et al., 2015)、我々の方法がこの能力を持ち得ることを期待する。セクション 4.1.3 での更なる分析も、我々の仮説が妥当であることを示す。したがって、我々は、連続的な行動空間を持つタスクにおいても、小さなK（例えば、4または8）を採用する。

パラメータ更新の方法 評論家ネットワークは全エージェントの観測と行動を考慮したので、ネットワークの出力（すなわち、実Q値Qi）はQi（hoi , ~o-ii, ai , ~a-i ; wi）と表すことができる。したがって、式2、3、4をマルチエージェント式に拡張することができる。
(((10,11,12)))

実際には、集中学習と分散実行のパラダイムを採用する（Oliehoek et al. , 2008; Foerster et al., 2017; Lowe et al., 2017; Chu and Ye, 2017）を採用してモデルの訓練と展開を行うため、上式の情報を容易に収集することができる。その上、K-headモジュールとAttentionモジュールは集中評論家に埋め込まれたサブモジュールなので、バックプロパゲーションを用いてエンドツーエンドでエージェントの政策と共同で最適化することができる。

3.4 the discussion

我々の注意力批判者は、チームメイトの動的な共同方針を適応的に明示的にモデル化する大きな能力を持っています。これは3つの観点から理解することができる。

第一の観点は、共同政策である。式8は、Σ K k=1Wk i (wi) ≡ 1を確実にするので、Wi(wi)は特定の共同政策の確率分布 ~π-i(A~-i |s) を表すことができなければならない。

第二の観点は、適応的な方法である。すなわち、Wi(wi)はチームメイトの動的な政策に適応的に反応することができる。これは，行動条件付きQ値Qk i (s, ai |~a-i ; wi)はエージェント・チームのすべての行動を考慮しているので，その値は現在の~π-i に依存しない経験タプル (s,hai , ~a-ii, ri , s0 ) , (hoi , ~o-ii,hai , ~a-ii, ri ,ho 0 i , ~o0 -i i) を使って推定できるためである．これは、〜π-i が変化しても、Qk i は値を変える必要がない（なお、Qk i は学習する必要がある）ことを意味する。安定した Qk i があれば、注目重み Wi(wi) は異なる ~π-i に容易に適応でき、エージェントは素早く政策を調整することができる。

最後の観点は、評論家ネットワークが数学的分析に基づいて設計されており、式7を明示的に近似するための特別な構造を導入していることである。これは有名なDueling Network (Wang et al., 2015) と同様であり、Q値を優位性とベースラインの和として明示的に近似する（すなわち、Q(s, a) = A(s, a) + V (s) ）ものである。これに対し、先行研究（Foerster et al., 2017; Mao et al., 2017; Lowe et al., 2017）のように完全連結型ネットワークを用いて集中型クリテックを実装した場合、完全連結型クリテック・ネットワークがこのような綿密なタスクを達成することは困難であろう。

4 experiment

実験は以下の設定に基づき行われる。批評家はデフォルトで 4head attention network を採用する。アクターは2つの隠れ層を持つフィードフォワードネットワークを用いる。批評家、アクターともに、各隠れ層は 32 個のニューロンを持つ。その他のハイパーパラメータは以下の通りである：アクターの学習率は 0.001、クリティックの学習率は 0.01、ターゲットネットワークの学習率は τ = 0.001、再生バッファサイズは 100000、バッチサイズは 128、割引率は γ = 0.95 である。ネットワークの構成は付録の通りである。

4.1 the packet routing environment

環境の説明 インターネット上でパケットをルーティングするためのよりよい方法を見つけ出すことが我々のグループの研究テーマであるため、ルーティング・タスクで我々の手法を評価する。図4に示すように、小さなトポロジーはインターネット・トラフィック・エンジニアリングのコミュニティで最も古典的なものであり (Kandula et al., 2005)、大きなトポロジーは我々のアプリケーションで実際に使われているトポロジーである。各トポロジでは、いくつかのエッジルータが存在する。各エッジルータは、利用可能なパスを通じて他のエッジルータに送信されるべき集約されたフローを持っている（たとえば、図4（a）では、BはDにフローを送信するように設定されており、利用可能なパスはBEF DとBDである）。各経路は複数のリンクで構成され、各リンクにはリンク使用率があり、これはこのリンクの最大フロー伝送能力に対するこのリンクの現在のフローの比率に等しい。ルーター間の協力の必要性は次のとおりである：1つのリンクは複数のルーターからのフローを伝送するために使われることがあるので、ルーターは同時に同じリンクに多すぎたり少なすぎるフローを分割してはならない；さもなければこのリンクは過負荷か低負荷のどちらかになってしまう。

問題の定義 ルータは我々のアルゴリズムによって制御され、ネットワーク全体の最大リンク使用率 (MLU) を最小にするために、良いフロー分割方針を学習しようとする。この目標の背後にある直観は、高いリンク利用率はバースト的なトラフィックを扱うのに望ましくないということである。観測には、ルータのバッファ内のフロー要求、最新の10ステップの推定リンク使用率、およびルータがとった最新のアクションが含まれる。アクションは各利用可能なパスの分割比率である。報酬はMLUを最小化したいので、1-MLUである。ローカルリンク利用率に基づく探索ボーナスは適宜追加することができる。

ベースライン MADDPG (Lowe et al., 2017) とPSMADDPGV2 (Chu and Ye, 2017) は、連続したアクション空間を持つ分散タスクを扱える最先端のRLベース手法であるため、ベースラインとして採用する。また、チームメイトの情報を収集するために中央集権的な批判を適用しているが、注意のメカニズムはない。MADDPGは完全連結ネットワークを用いて集中批判者を実装し、PSMADDPGV2はパラメータ共有法（批判者ネットワークの一部を他のエージェントと共有する）を用いて非明示的に他のエージェントをモデル化する。さらに、ルールベースのWCMPとKhead-MADDPGを比較する。WCMP（Zhou et al., 2014; Kang et al., 2015）は、Equal-Cost Multi-Path routing algorithm9 の Weighted-Cost 版で、現実のルータに適用されている最も一般的なマルチパス・ルーティング・アルゴリズムである。Khead-MADDPGは、K-head Moduleの分岐を直接マージして実Q値を生成するアブレーションモデルであり、このモデルにはアテンション機構は存在しない。

4.1.1 simple case test and scalability test

20の独立した実験の平均報酬を図5と図6に示す。見ての通り、小さなトポロジーの場合、ATT-MADDPGはMADDPGやPSMADDPGV2より多くの報酬を得ることができるが、Khead-MADDPGモデルは全く機能しない。これは、良い結果を得るためには、Kヘッドモジュールとアテンションモジュール（単一のKヘッドモジュールではない）の組み合わせが必要であることを意味しています。PSMADDPGV2の性能は満足のいくものではないことが判明したが、これはエージェントの異質性に起因すると思われる。

大きなトポロジーの場合、ATT-MADDPGは他の手法よりも大きなマージンをもって性能を発揮する。これは、ATT-MADDPGがより優れたスケーラビリティを持っていることを示している。その理由として、Attention ModuleがQ値推定をより関連性の高いエージェントの行動に注目させることができる（その結果、無関係なエージェントの影響が弱まる）ためであると考えられる。図4(b)を例にとると、エージェント4はエージェント3よりもエージェント1やエージェント2に注目する可能性が非常に高い。この性質により、ATT-MADDPGはエージェント数が増加するような複雑な環境下でもうまく機能する。一方、エージェントを明示的にモデル化する機構がない場合、MADDPGはこのようなスケーラビリティを備えていない。

どちらのトポロジーにおいても、1000エピソードを学習した後、ATT-MADDPGはルールベースのWCMPより優れた性能を示す。これは、RLベースのATT-MADDPGが将来の行動の影響を考慮できるため、高いレベルでの協調を達成するのに有利であるのに対し、ルールベースのWCMPは現在の行動の影響しか考慮できないためである。

4.1.2 robustness test

ATT-MADDPGでは特殊なハイパーパラメータKを導入しており、Kの設定が性能にどのように影響するかを調べることが必要不可欠である。前述したように、上記の結果はK = 4のときに得られたものである。さらにKを2、8、12、16に設定し、同様の実験を行った。20回の独立した実験の平均報酬を図7と図8に示す。見てわかるように、小さなトポロジーの場合、Kが大きくなるにつれて得られる報酬は増加し、Kを8としたときに大きな増加が見られる。大きなトポロジーの場合、Kを16に設定すると、小さな増加が観察される。全体として、ATT-MADDPGはすべての設定において、MADDPGよりも多くの報酬を得ることができる。その結果、ATT-MADDPGは広い範囲のKでロバスト性を維持し、良い結果を得ることができると結論付けることができる。

4.1.3 further study on k-head and attention

3.3 節では、注目重み Wk i (wi) は確率 ~π-i(~a-i|s) を近似するために用いられ、K-head Module は類似した ~a-i をグループ化する能力を持つと主張した。本実験では、上記の主張が実験結果と整合しているかどうかを検証したい。具体的には、リプレイバッファから3000個の経験タプル（s, a, Q(s, a)）をランダムにサンプリングし、異なるヘッドのQ値と30個の非桜狩りサンプル10の注目重みを図9に示す。見ての通り、head4は最も滑らかなQ値を持ち、head4の重みは他のheadの重みに比べ非常に大きい。一方、head1はQ値の変動幅が大きく、head1の重みはかなり小さくなっている。

以上の現象から、K-head Moduleは確かに似たような〜a-iをグループ化できると考えられる。例えば、重みの大きいhead4は非重要な〜a-iの大きな集合（例えば、フロー分割比が［0.3、0.7］の間）を表し、重みの小さいhead1は重要な〜a-iの小さな集合（例えば、フロー分割比が［0.8、0.9］）を表す可能性があります。その説明は以下の通りである。Q値の観点からは、head4は非重要な〜a-iを表していると考えられるので、ほとんどの局所行動aiはMLUに大きな影響を与えない（従って、報酬とQ値も大きくならない）ので、head4が滑らかなQ値を持つのは合理的である。注目重量の観点からは、頭部4は多くのルータに好まれている非巡礼的な〜a-iの大きな集合を表す可能性があるので、頭部4がグループ化した〜a-iの確率和Σ〜a-i〜π-i（〜a-i｜s）は大きくなり、注目重量は確率〜π-i（〜a-i｜s）の近似であると考えると、頭部4が他の頭部よりも大きな注目重量のあることは合理的になるだろう。head1のQ値と注目度を同様に分析することで、我々の仮説（K-head Moduleが類似の〜a-iをグループ化できる）が合理的であることを示すことができる。

4.2 the benchmark environment

MADDPGでも採用されている2つのベンチマーク環境について考えてみる。それらを図 10 に示す。

協調航法(Co. Na.). 10×10の2次元平面のランダムな位置に、3つのエージェントと3つのランドマークが生成される。エージェントは我々のアルゴリズムによって制御され、全てのランドマークを協調的にカバーしようとする。観測は他のエージェントとランドマークの相対的な位置と速度である。アクションは速度である。報酬は各ランドマークに対する任意のエージェントの負の近接度である。

プレデター・プレイ（Predator Prey）（Pr.Pr. 10×10の2次元平面のランダムな位置に3匹の捕食者と1匹の餌食者が生成される。捕食者は我々のアルゴリズムによって制御され、協調的に獲物を捕らえようとする。観測と行動は、協調航法環境と同じである。報酬は、任意の捕食者が獲物に負の接近をすることである。また、捕食者は獲物を捕らえると10の報酬を得ることができる。

ベースライン MADDPG, PSMADDPGV2, Khead-MADDPG の他に、GreedyPursuit と呼ばれるルールベースの手法との比較：協調航法の場合、エージェントは常に最も近いランドマークに行き、捕食者の場合は、常に捕食者の現在地に行く。

結果 50の独立した実験の平均的な最終安定報酬を表2に示す。パケットルーティング環境での結果とは対照的に、現在の環境ではPSMADDPGV2がMADDPGよりもうまく機能している。これは、現在の環境ではエージェントが同質であり、パラメータ共有法がより効率的であるためと考えられる。さらに、ATTMADDPGはMADDPGやPSMADDPGV2よりも多くの報酬を得ることができる。これは、本手法が一般的に適用可能であり、良好な性能を有していることを示している。GreedyPursuitは、チームメイトが同じランドマークへ行くことや、獲物がランダムに別の場所へ逃げることを考慮していないため、パフォーマンスが低下しています。Khead-MADDPGは、収束がうまくいかず、ランダムなエージェントになることがあり、さらに悪い挙動を示す。

政策分析 図11は、協調航行タスクにおいてATT-MADDPGが学習した収束的共同方針を示したものである。最初（すなわち、最初の絵）、A1とA2は最も近いランドマークL2を共有し、一方、A3はL1とL3に非常に近い。したがって、A1はL1とL2の中心へ、A2はL2とL3の中心へ、A3はL1とL3の中心へとためらいながら移動する。いくつかのタイムステップの後、状態は2番目の絵に変化する。このとき、A2とA3は、A1がL1に行くことを理解している。したがって、A2はL2へ、A3はL3へ、A1はL1へと、次のタイムステップで直接移動する（つまり、中央の3枚の絵）。その結果、エージェントは最後の写真に示すように、すべてのランドマークをカバーするようになりました。これらの行動は、エージェントが本当に協調的な共同方針を学習したことを示している。

5 related work

エージェントモデリングは、相互作用の履歴に基づいて他のエージェントのモデルを構築するプロセスである。モデルには、信念、政策、行動、クラス、目標など、関心のあるあらゆる性質が含まれる（Albrecht and Stone, 2018）。これまでの手法の多くは、ゲーム理論（Ganzfried and Sandholm, 2011）やグリッドワールドの設定に基づいており、ネットワークパケットルーティングのような実世界のアプリケーションにはほとんどスケールしていない。

近年、DRLに基づく手法が大規模な問題に対するエージェントモデリングを行うために研究されている。我々の手法はそのような手法の一例であり、最も関連する研究は、DRON (He et al., 2016), DPIQN (Hong et al., 2018), LOLA (Foerster et al., 2018), SOM (Raileanu et al., 2018), Mean Field Reinforcement Learning (MFRL) (Yang et al., 2018) である。DRONは、相手の行動をエージェントの政策ネットワークに埋め込む。このように、相手の行動はエージェントの政策の隠れ変数と見なすことができる。この隠れ変数がどの程度政策に影響を与えるかを制御するために、別のゲーティングネットワークが用いられる。DPIQNはDRONと非常によく似ています。制御可能なエージェントのDQNに協力者の政策機能を埋め込み（Mnih et al.，2015）、協力行動を生成できるようにする。  

LOLAでは、エージェントのポリシー更新ルールに追加項が明示的に含まれている。この追加項は、他のエージェントへの影響を考慮することができる。SOMはすべてのエージェントに対して共有の政策ネットワークを学習する．政策ネットワークの入力には、異なるエージェントを区別するためのゴールフィールドが含まれる。著者らは，政策ネットワークがエージェントの行動をある程度モデル化できることを見出した．MFRLは、複数のエージェント間の相互作用を、一つのエージェントと他のチームメイトの平均効果との間の相互作用によって近似的にモデル化する。これらのDQNに基づく手法が離散的な行動空間を持つタスクに対して集中的な政策を学習するのとは対照的に，我々の手法は連続的な行動空間を持つタスクに対して分散的な政策を生成することができる．いくつかのDQNに基づく手法（Sunehag et al., 2017; Rashid et al., 2018）は分散化政策を生成でき、ベースラインのMADDPG（Lowe et al., 2017）とPSMADDPGV2（Chu and Ye, 2017）は連続的行動空間を持つ分散化政策を訓練できるが、それらは他のエージェントのモデルを効率的に構築しない、代わりにクレジット割り当て、競争エージェントなど他の問題に対処している。より多くの関連研究を付録に示す。

6 conclusion

本論文では、協調的な分散マルチエージェント環境において、チームメイトの政策をモデル化し、利用するための新しいアクター・クリティックRL手法を提案する。本手法は、注意メカニズムを集中型批評家に組み込み、チームメイトの動的な共同方針を適応的に明示的にモデル化する特別な構造を導入している。その結果、全てのエージェントが効率的に協力し合うようになる。さらに、本手法は、連続的な行動空間を持つ分散タスクを扱うための分散化された政策を学習することができる。

我々は、ベンチマークタスクと実世界のパケットルーチングタスクの両方で本手法を評価した。その結果、本手法は最先端のRLベース手法やルールベース手法を大きく上回るだけでなく、良好なスケーラビリティとロバスト性を達成することがわかった。さらに、我々の手法をより深く理解するために、徹底的な実験も行っている。(1)アブレーションモデルにより、提案モデルの全ての構成要素が必要であることが示された。(2)Q値と注意重みの研究により、我々の手法が洗練された注意メカニズムを実際に習得したことが示された。(3)具体的な政策の分析により、エージェントが本当に協調的共同政策を学習したことが示された。

今後は、行動空間が離散的で、競合するエージェントが存在する環境にも、本手法を拡張していく予定である。