* # Ceph 相關筆記
## Ceph的組成
基本的 Ceph 包含 Monitor 節點與 OSD 節點。
- Monitor node
  - 主要的職責在於收集並記錄當前的各種叢集資訊
- OSD node
  - 負責儲存物件
- MDS node(僅在提供 Ceph File System 時使用)
-----------
## Ceph 儲存資料方式
當user要求寫入一個物件時，Ceph會依照user要求寫入的Pool以及物件的相關資訊，透過CRUSH演算法決定要將物件放在哪一個PG中，然後讓user直接跟擁有該PG的OSD node進行傳輸。

在Ceph的架構中，每個OSD node都負責儲存user上傳的物件，但這些物件實際上是被存放在PG中。

### Ceph Placement Group(PG)
PG是Ceph實際存放資料的單元，每個PG又都隸屬於一個Pool，而一個Pool可以擁有多個PG。
A Placement Group (PG) is a logical collection of objects that are replicated on OSDs to provide reliability in a storage system. Depending on the replication level of a Ceph pool, each PG is replicated and distributed on more than one OSD of a Ceph cluster. You can consider a PG as a logical container holding multiple objects, such that this logical container is mapped to multiple OSDs:
![](https://i.imgur.com/7jrhabI.png)
參考網站：https://www.oreilly.com/library/view/ceph-designing-and/9781788295413/b8902238-fd69-4c48-8df4-df95e34ca098.xhtml

------
Pool 可以設定資料的複製量（Data Replication），以確保當某個 OSD 節點或 OSD Daemon 故障時，使用者的資料皆能被恢復。
物件在寫入至 PG 內後，PG 會依照所屬 Pool 設定的複製數目，將物件複製到其他位於不同 OSD 節點的 PG 上。
![](https://i.imgur.com/yTjWPFc.png)
圖片轉載自 [HOW :: Data is Storage Inside Ceph Cluster](http://karan-mj.blogspot.com/2014/01/how-data-is-stored-in-ceph-cluster.html)

--------
以上圖為例，Pool - A 設定的複製數目為 3，因此粉紅色的物件會被複製到三台 OSD 節點上。
基於複製的概念，我們也可以推導出若有一個 Pool 設定的複製數目為 3，則同時有超過三個 OSD Daemon 發生永久性故障時，有可能導致部分資料永遠無法恢復，因為若某個物件的所有複製剛好都落在這三個 OSD Daemon，就無法救援資料了。

參考自:
1. https://jimwayne.blogspot.com/2015/04/cephceph.html
2. 