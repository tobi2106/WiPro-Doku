Im folgenden stehen die Fragen und Antworten, die wir im Expertengespräch gestellt haben. 
Die Antworten wurden während des Gespräches mitgeschrieben. 

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

---

!!! question
    Sind Constrains wie: CPU-Auslastung, Speicher-Auslastung und verfügbarer Speicherplatz ausreichend um ein Workload sinnvoll auf einem Node neu zu verteilen?

Antwort:

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

Antwort:

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

---

!!! question
    Zum Abschluss haben wir die KI nach eine Frage zu Kubernetes Scheduling gebeten. 
    Passend zu der `Woche der KI` hat sich das LLM folgende Frage formuliert:
    <br>
    How can machine learning be integrated into Kubernetes scheduling for better decision-making?

Antwort: 

---

Vielen Dank für das Gespräch!
