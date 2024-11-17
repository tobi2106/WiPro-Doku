Im folgenden stehen die Fragen und Antworten, die wir im Expertengespräch gestellt haben. 
Die Antworten wurden während des Gespräches mitgeschrieben. 


Descheduler Stand Kratzke:
- Tags ans Pods RequiredDuringScheduling und RequiredDuringRuntime

# Szenario mit zustandslosen Workloads

Outline:<br>
Wir haben ein Kubernetes System mit zustandslosen Workloads. 
Die Workloads sind auf verschiedene Nodes verteilt. 
Verteilt werden die Workloads über den standard Kubernetes Scheduler. 
Ein Rescheduler soll bereits verteilte Workloads basierend auf den dynamischen Zustand auf den Cluster neuverteilen (nach einem Trigger).
---
Dafür haben wir einen eigenen Scheduler geschrieben, 
der verschiedene Strategien für das Scheduling der Workloads verwenden kann - wie:<br>
- RoundRobin 
- Priority Scheduling
- First-Come, First-Served.
Es existiert ein `MonitoringAgent` der alle Workloads beobachtet und deren Metrics analysiert.  


!!! question
    Wenn das Problem des Scheduling von Workloads allgemein betrachtet wird, muss das `Bin-Packing Problem` gelöst werden. 
    In Kontext zu Betriebssystemen müssen verschiedene Trade-offs betrachtet werden. 
    So muss darauf geachtet werden, 
    dass Prozesse fair und effizient auf verfügbare Ressourcen verteilt werden. 
    Dabei spielen Aspekte wie Antwortzeiten, 
    Durchsatz und Fairness eine zentrale Rolle. 
    <br>
    Inwiefern lässt sich das Konzept von Fairness aus traditionellen Scheduling-Ansätzen auf Kubernetes-Umgebungen übertragen, und 
    welche speziellen Herausforderungen entstehen dabei im Hinblick auf die dynamischen und 
    verteilten Workloads von Cloud-nativen Anwendungen?


Antwort:
- Ganz ehrlich: Keine Ahnung, das legt der Scheduler fest und kann so nicht gesagt werden. Workload der auf binpacking läuft, wahrscheinlicher für Systemausfälle (viele Pods auf Node, Stecker ziehen).  


---

!!! question
    Sind Constrains wie: CPU-Auslastung, Speicher-Auslastung und verfügbarer Speicherplatz ausreichend um ein Workload sinnvoll auf einem Node neu zu verteilen?

Antwort:
- Kommt drauf an (auf die Strategie). CPU: Häufig null aussagekraft weil
Webservices häufig wenig Auslastung weil wegen warten auf IO --> CPU Metrik schlägt viel zu spät an. Nicht sinnvoll
Batch-Job Neuronales Netz: CPU-lastig/GPU-lastig (lange so). CPU sinnvoll. 
Kommt drauf an, was wir machen.
Google-Paper Workload-Kategorien empirisch in Google-Rechenzentren. Cloud-Prog Scheduling zu finden.
Kubernetes default Scheduler häufig ür Webservices

---

!!! question
    Im Kubernetes System gibt es den `Metrics-Server` der für das Autoscaling von Workloads zuständig ist. 
    Unter anderem bietet der Metrics-Server die Möglichkeit über die `Metrics API` auf Ressourcen von Pods und Nodes zuzugreifen. 
    Wäre es sinnvoll, 
    diese Metriken auch für das Rescheduling heranzuziehen, 
    oder sind dafür zusätzliche Metriken und Datenpunkte notwendig? 
    Inwiefern könnten die Informationen wie historische Daten, 
    Netzwerk-Latenzen oder spezifische Workload-Anforderungen dabei helfen das Scheduling zu verbessern?

!!! note
    Kann Metrics-Server überhaupt weitere (für Rescheduling) sinnvolle Metriken liefern, die nicht rein über Metrics-API auch schon so bekommen werden können?


Antwort:
Metrics-Server liefert sicherlich Node-Metriken. Mal anschauen. Vllt gibt der noch Aufschluss darüber. Ist-Zustand des Systems. Zukunft vorhersage schwierig (bei periodischen Jobs evtl. einfacher).

---

!!! question
    Ein Kubernetes Cluster wird unter Anderem oft dafür genutzt, um die Verfügbarkeit von Anwendungen sicherzustellen. 
    Aus Sicht des Cluster-Betreibers möchte man Einerseits eine hohe Ressourcenauslastung erreichen. 
    Andererseits möchte man sicherstellen, dass die Verfügbarkeit der Workloads nicht beeinträchtigt wird. 
    Worauf sollte man beim Rescheduling achten, um diesen Trade-off zwischen Ressourcenauslastung und Verfügbarkeit zu lösen?
    <br>
    Gibt es Strategien oder Metriken die dabei helfen können, um diesen Konflikt zu lösen?
    <br>
    Hat die Auslastung von Nodes irgendeinen Einfluss auf Antwortzeiten von Workloads darauf oder so?

Antwort:
Das ist spannend! Letztendlich definieren wir eine Zielfunktion, die wir maximieren wollen. 

Schedulen auf 2 Ebenen Pods im Cluster und Cluster himself zu skalieren. Clusterskalierung läuft deutlich langsamer als Workload-Skalierung in Cluster (Cluster locker mal 10 Minuten, Workloads im Sekundenbereich)


**Für Zusatzfrage**

Nodes die wirklich ausgelastet sind (Ressourcenmäßig) sind langsamer (Betriebssysteme RAM, Kontextwechsel kostet Zeit). Request für Scheduler entscheidend.
Limits sind wichtig für Auslastung der Nodes, um Monopolisierung zu vermeiden (Fairness).
Cloudprogramming: Mesos hat Fairness-Definition drin im Skript. Anschauen. Priorisierung. Ressourcen werden beliebig elastisch an Projekte verteilt.


# Szenario mit Cloud-Native Anwendungen

Outline:<br>
Das Szenario wird aus Sicht eines Kubernetes Providers betrachtet. 
<br>
Kubernetes bietet die Möglichkeit einen sogenannten `Descheduler` zu nutzen. 
Dieser ist kein Default von Kubernetes kann jedoch einfach mit über einen `helm` zum Cluster hinzugefügt werden.
Der `Descheduler` wird eingesetzt, um die Verteilung von Pods in einem Cluster nachträglich zu optimieren, 
wenn zum Beispiel die Bedingungen im Cluster sich geändert haben. 
Da Kubernetes Cluster jedoch dynamisch sind und der Cluster-Zustand sich ändert (darunter Ressourcenbedarf, die Auslastung und die Verfügbarkeit von Nodes) kann es erforderlich sein auch bereits laufende Pods neu zu verteilen. 
<br>
<br>
Beim `Descheduler` werden folgende Szenarien betrachtet:<br>
- Ungleichmäßige Auslastung:<br>Wenn Nodes über- bzw. unterausgelastet sind
- Veränderte Bedingungen:<br>Wenn Labels, Taints und Affinitätsregeln, die ursprünglich zur Platzierung geführt haben, nicht mehr zutreffen
- Node-Ausfällen:<br>Wenn Nodes ausfallen und deren Pods neu verteilt wurden. 
Der `Descheduler` übernimmt nicht selbst das erneute Scheduling, sondern überlässt dies dem Standard-Scheduler. 


!!! question
    Der Descheduler wird verwendet, um Workloads im Kubernetes Cluster effizient auf verschiedene Nodes neu zu verteilen, wenn sich die Cluster-Bedingungen geändert haben. Dies kann notwendig sein, um Ressourcen effizienter zu nutzen, Lasten gleichmäßiger zu verteilen und die Verfügbarkeit zu verbessern. Im Betracht von Cloud-Nativen Anwendungen werden unter andrem jedoch zustandsbehaftete Workloads genutzt. 
    <br>
    Gibt es eine Möglichkeit zustandsbehaftete Workloads auf einem Cluster neu zu Verteilen ohne, 
    dass die Nutzer des Clusters benachteiligt werden?

Antwort:
NoSQL Datenbank Sync (Leader-Election- Algorithmen (N+1/2 (Raft)))

---

!!! question
    Der Anwendungsbereich ist so klein, dass nur Provider es interessiert, 
    wie die Pods gescheduled werden. 
    Entsprechend können diese aber keine Vorhersage treffen, 
    ob die Nutzer cloud native Anwendugen auschließlich auf den Pods laufen lassen. 
    Anderes andres ausgedrückt, 
    ob die Nutzer Anwendungen laufen lassen, 
    welche sich für das Descheduling anbieten würden.
    <br>
    Was wären Use-Cases in denen Sie solch einen `Descheduler` in Betracht ziehen würden?

**Antwort:**
Problemlos bei Stateless, kann Problem bei Stateful werden. Rolling-Update von Kubernetes, kann man auch gut verwenden für unseren Rescheduler.

Für jede Anwendung Schmerzgrenze, **wieviele Ressourcen darf Kubernetes mir abziehen? Was meine Schmerzgrenze?**
Stateful sehr schwierig, aufwändig. untere Grenze ist immer einfach. Obere Grenze ist schwieriger zu definieren.

---

!!! question
    Obwohl der `Descheduler` von Kubernetes entwickelt und verwaltet wird, 
    gehört er nicht zur Standartinstallation und 
    wird nur als als optionale Erweiterung angeboten. 
    <br>
    Warum ist der Descheduler kein fester Bestandteil des Kubernetes-Standards? 
    <br>
    Welche technischen oder strategischen Überlegungen könnten dahinterstehen?“

Antwort: 

Bringt Unsicherheiten und Schwierigkeiten ins System. bestreben einen stabilen Zustand zu erreichen, Rescheduling stört.
Lohnt sich häufig

Workloads unterbrechen können nicht alle (schlecht) desginten ab. Bei Stateless Anwendungen haben wir häufig nicht Probleme wie bei Stateful-Sets. Wird Datenbank verwendet, die kein Raft verwendet? Paxos? Irgendeine untere Grenze Definiert, unter der es nicht mehr funktioniert?


---

!!! question
    Zum Abschluss haben wir die KI nach eine Frage zu Kubernetes Scheduling gebeten. 
    Passend zu der `Woche der KI` hat sich das LLM folgende Frage formuliert:
    <br>
    How can machine learning be integrated into Kubernetes scheduling for better decision-making?

Antwort: 
Zeitreihenprognosen die man nutzen kann. Workloads Periodizitäten unterworfen (Montags morgen Büro = Peak). Kann man
aus Historie ableiten, wann er welche Peaks haben wird? Klassisches Machine Learning Problem. Automatisierte Cluster-Skalierung --> Ab Feierabend bspw. kann wieder runterskaliert werden.
Machine Learning kann man das anpassbarer machen.
 
---

Vielen Dank für das Gespräch!
