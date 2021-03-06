.. _pcflow_secure_fiesta:

-------------------------------
Flowを使用したアプリケーションの保護
-------------------------------

Flowは、Nutanix AHVおよびPrism Centralに統合されたアプリケーション中心のネットワークセキュリティ製品です。
AHV上で動作するVMに対して、豊富なネットワークトラフィックの可視化、自動化、およびセキュリティを提供します。

マイクロセグメンテーションはシンプルなポリシーベースの管理を使用してVMネットワーキングをセキュアにするFlowの機能です。
Prism Centralのカテゴリ（論理グループ）を使用して、強力な分散型ファイアウォールを作成することができます。また、Calmと組み合わせることで、作成したアプリケーションの安全性が確保された自動展開が可能になります。

**この演習では、Fiestaアプリケーションへのアクセスを制限し、アプリケーション層間のトラフィックを保護します。**

Fiestaアプリケーションの保護
+++++++++++++++++++++++

Flowは、仮想マシンを迅速にグループ化するために使用されるAppType、AppTier、Environmentなどの複数のシステムカテゴリを提供しています。
セキュリティポリシーは、これらのカテゴリを使用して適用されます。
これらの既存のカテゴリをそのまま使用することも、カスタムグループ化のために独自のカテゴリを追加することもできます。


カテゴリーの定義
........................

Prism Centralを使用して、カテゴリをメタデータとしてVMにタグ付けやポリシーがどのように適応されるべきか決定します。

#. **Prism Central** の :fa:`bars` **> 仮想インフラ(Virtual Infrastructure) > カテゴリ(Categories)** と進みます。

#. **AppType** にチェックを入れ **(画面上部の)アクション(Actions) > Update** をクリックします。

   .. figure:: images/12.png

#.  :fa:`plus-circle` アイコンをクリックして、カテゴリに値を入力します。

#. 値の名前として *Initials*-**Fiesta** を指定します。

   .. figure:: images/13.png

#. **保存(Save)** をクリックします。

#. 続いて、**AppTier** にチェックを入れ **(画面上部の)アクション(Actions) > Update**  をクリックします。

#. :fa:`plus-circle` アイコンをクリックして、カテゴリに値を入力します。

#. 値の名前として *Initials*-**Web** を指定します。このカテゴリはWebアプリケーション層を想定して適用します。

#. 続けて :fa:`plus-circle` アイコンをクリックして、カテゴリに値を入力します。値の名前として *Initials*-**DB** を指定します。
   このカテゴリはMySQLデータベース層を想定して適用します。

   .. figure:: images/14.png

#. **保存(Save)** をクリックします。

セキュリティポリシーの作成
..........................

Nutanix Flowには、ネットワーク中心のアプローチではなく、ワークロード中心のアプローチを使用するポリシー駆動型のセキュリティフレームワークが含まれています。
そのため、VMのネットワーク構成がどのように変化しても、またデータセンターのどこに存在していても、VMへのトラフィックとVMからのトラフィックを精査することができます。また、ワークロード中心のネットワークに依存しないアプローチにより、仮想化チームはネットワークセキュリティチームに頼ることなく、これらのセキュリティポリシーを実装することができます。

セキュリティポリシーはカテゴリ毎に適用され、VM自体には適用されません。
したがって、与えられたカテゴリ内でどれだけの VM が起動されているかは問題ではありません。
カテゴリ内の VM に関連付けられたトラフィックは、どの規模でも管理者の介入なしに安全に保護されます。

Fiestaアプリケーションを保護するセキュリティポリシーを作成します。

#.  **Prism Central** の :fa:`bars` **> ポリシー(Policies) > セキュリティポリシー(Security Policies)** をクリックします。

#. **セキュリティポリシーの作成(Create Security Policy) > セキュアアプリケーション(セキュリティポリシー) (Secure Applications (App Policy)) > 作成(Create)** をクリックします。

#. 次のフィールドに入力します。

   - **名前(Name)** - *Initials*-Fiesta
   - **目的(Purpose)** - Restrict unnecessary access to Fiesta
   - **このアプリを保護(Secure this app)** - AppType: *Initials*-Fiesta
   - **カテゴリ別にアプリタイプにフィルターをかけます** に **チェックを入れない** Do **NOT** select **Filter the app type by category**.

   .. figure:: images/18.png

#. **次へ(Next)** をクリックします。

#. もしチュートリアルのプロンプトが表示されたら、**分かりました！(OK, Got it!)** をクリックします。

#. セキュリティポリシーをより詳細に設定するには、アプリケーションのすべてのコンポーネントに同じルールを適用するのではなく、**代わりにアプリ階層にルールを設定します(Set rules on App Tiers, instead)** から行います。

   .. figure:: images/19.png

#. **+階層を追加(+ Add Tier)** をクリックします。

#. **AppTier:**\ *Initials*-**Web** をドロップダウンから追加します。

#. Steps 7-8 を同様に繰り返し、 **AppTier:**\ *Initials*-**DB** も追加します。

   .. figure:: images/20.png

   次に、アプリケーションとの通信を許可するソースを制御する **Inbound** ルールを定義します。
   すべての受信トラフィックを許可することも、ホワイトリストのソースを定義することもできます。

   このシナリオでは、すべてのクライアントからTCPポート80 Web層への インバウンドトラフィックを許可します。

#. **インバウンド Traffic(Inbound Traffic)** の **移行元を追加(+ Add Source)** をクリックします。

#. 次のフィールドを入力し、全ての受信IPアドレスを許可します。

   - **～で移行元追加:(Add source by:)** - Select **サブネット/IP(Subnet/IP)**
   - **0.0.0.0/0**

   .. note::

     ソースはカテゴリで指定することもでき、この値はネットワークの場所変更に関係なくVMを追跡できるため、より柔軟性が高くなります

#. インバウンドルールを作成するために、 **AppTier:**\ *Initials*-**Web** 左側に表示される **+** アイコンをクリックします。

   .. figure:: images/21.png

#. 次のフィールドに入力します。

   - **Protocol** - TCP
   - **Ports** - 80

   .. figure:: images/22.png

   .. note::

     1つのルールに複数のプロトコルとポートを設定できます。

#. **Save** をクリックします。

   Calm は、スケールアウト、スケールイン、アップグレードなどのワークフローのために VM へのアクセスすることもあります。
   Calm は、TCP ポート 22 を使用して SSH 経由でこれらの VM と通信します。

#. **インバウンド Traffic(Inbound Traffic)** 下の **+移行元を追加(+ Add Source)** をクリックします。

#. 次のフィールドに入力します。

   - **～で移行元追加:(Add source by:)** - Select **サブネット/IP(Subnet/IP)**
   - *Prism Central IP*\ /32

   .. note::
     **/32** は範囲ではなく、単一のIPを示します。

   .. figure:: images/23.png

#. **追加(Add)** をクリックします。

#. **AppTier:**\ *Initials*-**Web** の左側に表示される **+** アイコンをクリックし、 **TCP** , ポート番号 **22** を指定して、**保存(Save)** をクリックします。

#. **AppTier:**\ *Initials*-**DB** に対してもステップ18と同様の操作を繰り返し、CalmがデータベースVMと通信出来る様にします。

   .. figure:: images/24.png

   デフォルトでは、セキュリティポリシーにより、アプリケーションはすべての送信トラフィックを任意の宛先に送信できます。アプリケーションに必要な唯一のアウトバウンド通信は、DNSサーバーとの通信です。

#. **アウトバウンド Traffic(Outbound Traffic)** 下の **ホワイトリストのみ(Whitelist Only)** を選択し、 ドロップダウンメニューから **+デスティネーションを追加(+ Add Destination)** を選択します。

#. 次のフィールドに入力します。

   - **ディスティネーションの追加(Add destination by:)** - **サブネット/IP(Subnet/IP)** を選択
   - *ドメインコントローラのIP*\ /32

   .. figure:: images/25.png

#. **追加(Add)** をクリックします。

#. **AppTier:**\ *Initials*-**Web** の右側に表示される **+** を選択して **UDP** ポート **53** を指定して、 **Save** をクリック することで
   DNSのトラフィックを許可します。同様に **AppTier:**\ *Initials*-**DB** に対しても行います。

   .. figure:: images/26.png

   アプリケーション層は他の層と通信を必要としポリシーはこのトラフィックを許可する必要があります。Webなどの一部の層は、同じ層内での通信を必要としません。

#. アプリケーション内の通信を定義するには、**Set Rules within App** をクリックします。

   .. figure:: images/27.png

#. **AppTier:**\ *Initials*-**Web** を選択し **No** をクリックして、この層内のVM間の通信を禁止します。層内に存在するWeb VMは1つのみです。

#. 続いて、**AppTier:**\ *Initials*-**Web** を選択した状態で **AppTier:**\ *Initials*-**DB** の :fa:`plus-circle` アイコンをクリックして、層間のルールを作成します。

#. 次のフィールドに入力します。Web層とDB層間で **TCP** ポート **3306** の通信を許可します。

   - **Protocol** - TCP
   - **Ports** - 3306

   .. figure:: images/28.png

#. **保存(Save)** をクリックします。

#. **次へ(Next)** をクリックして、ここまで設定してきたセキュリティポリシーを確認します。

#. **保存とモニター(Save and Monitor)** をクリックしてポリシーを保存します。


カテゴリ値の割り当て
.........................

ここで、以前に作成したカテゴリをFiestaブループリントからプロビジョニングされたVMに適用します。
フローカテゴリはCalmのブループリントの一部として割り当てることができますが、この演習の目的は、既存の仮想マシンへのカテゴリ割り当てを理解することです。

#. **Prism Central** の :fa:`bars` **> 仮想インフラ(Virtual Infrastructure) > 仮想マシン(VMs)** と進みます。

#. **フィルター(Filters)** をクリックし、 *Initials AHV Fiesta VMs* ラベルを選択して、仮想マシンを表示します。

   .. figure:: images/15.png

#. チェックボックスを使用して、アプリケーション(WebおよびDB)に関連付けれらた2つのVMを選択して、 **アクション(Actions) > カテゴリを管理(Manage Categories)** をクリックします。

   .. figure:: images/16.png

#. 検索バーで **AppType:**\ *Initials*-**Fiesta** を指定して、 **Save** アイコンをクリックします。

   .. figure:: images/16a.png

#. 続いて、*nodereact* VMのみを選択して **アクション(Actions) > カテゴリを管理(Manage Categories)** から **AppTier:**\ *Initials*-**Web** カテゴリーを指定し、 **Save** をクリックします。

   .. figure:: images/17.png

#. ステップ5 を繰り返し、MySQLのVMを **AppTier:**\ *Initials*-**DB** カテゴリーに指定します。

#. 最後に、Windows Tools VMに対して ステップ 5 の操作を行い **Environment:Dev** カテゴリーに指定します。

セキュリティポリシーの監視と適用
+++++++++++++++++++++++++++++++++++++++++

ポリシーを適用する前に、Fiestaアプリケーションが期待通りに機能していることを確認します。

アプリケーションのテスト
.......................

#. **Prism Central** の :fa:`bars` **> 仮想インフラ(Virtual Infrastructure) > 仮想マシン(VMs)** と進み、**-nodereact...** と **-MYSQL-...** 仮想マシンのIPアドレスをメモします。

#. *Initials*\ **-WinToolsVM** を起動します。

#. *Initials*\ **-WinToolsVM** のコンソールからブラウザを開き、 \http://*node-VM-IP*/ アクセスします。

#. アプリケーションが読み込まれ、タスクの追加および削除が出来ることを確認します。

   .. figure:: images/30.png

#. **Command Prompt** を開き、 ``ping -t MYSQL-VM-IP`` と実行し、クライアントとデータベース間で通信が出来ることを確認します。
　 コマンドを中断せず実行したままとします。

#. **Command Prompt** をもう1つ開き、``ping -t node-VM-IP`` と実行し、クライアントとWebサーバー間で通信が出来ることを確認します。
　 こちらも同様にコマンドを中断せず実行したままとします。

   .. figure:: images/31.png

Flowによる可視化
........................

#. **Prism Central** の :fa:`bars` **> 仮想インフラ(Virtual Infrastructure) > Policies > Security Policies >** と進み、 *Initials*-**Fiesta** をクリックします。

#. **Environment: Dev** がインバウンドソースとして表示されていることを確認します。ソースとラインは黄色で表示され、クライアントVMからのトラフィックが検出されたことを示します。

   .. figure:: images/32.png

   他に検出されたトラフィックフローがあった場合は、それらの線にマウスカーソルを合わせて使用中のポート情報を確認してみます。
   もし、表示されない場合は、スキップして次の項へ進みます。

#. **Update** をクリックして、ポリシーを編集します。

   .. figure:: images/34.png

#. **次へ(Next)** をクリックして、検出されたトラフィックフローが入力されるまで待ちます。

#.  **AppTier:**\ *Initials*-**Web** に接続する **Environment: Dev** 上にマウスオーバーし、 :fa:`check` アイコンをクリックします。

   .. figure:: images/35.png

#. **OK** をクリックし、ルールの追加を完了します。

   **Environment: Dev**  が青く表示されていれば、それがこのポリシーに含まれていることを示します。フローラインにマウスオーバーして、ICMP (ping トラフィック) と TCP ポート 80 の両方が表示されていることを確認してください。

#. **保存とモニター(Save and Monitor)** とクリックし、ポリシーを更新します。

Flowポリシーの適用
......................

定義したポリシーを適用するためには、ポリシーを **適当(Apply)** する必要があります。

#. *Initials*-**Fiesta** を選択し **アクション(Actions) > 適用(Apply)** をクリックします。

   .. figure:: images/36.png

#. 確認ダイアログに **APPLY** と入力し **OK** をクリックすることでポリシーが適用されます。

#. *Initials*\ **-WinToolsVM** コンソールに戻ります。

  クライアントからデータベースサーバーへの継続的なpingトラフィックはどうなりますか？このトラフィックはブロックされていますか？

#. クライアントVMがWebブラウザとWebサーバーのIPアドレスを使用してFiestaアプリケーションに引き続きアクセスできることを確認します。


まとめ
+++++++++

- マイクロセグメンテーションは、データセンター内から発生し、1台のマシンから別のマシンへと横方向に拡散する悪意のある脅威に対する追加の保護を提供します
- Prism Centralで作成されたカテゴリは、Calmのブループリント内で利用できます。
- セキュリティポリシーは、Prism Centralのテキストベースのカテゴリを活用します。
- フローは、AHV上で動作するVMの特定のポートやプロトコルのトラフィックを制限することができます。
- ポリシーはMonitorモードで作成され、ポリシーが適用されるまでトラフィックはブロックされません。これは、接続を学習し、意図せずにトラフィックがブロックされないようにするのに役立ちます。
