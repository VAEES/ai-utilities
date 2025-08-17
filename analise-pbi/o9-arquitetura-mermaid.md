flowchart LR
    O9Bucket["O9 Bucket"]
    CAPLeituraS3["CAP para Leitura do S3"]
    BaseHANAPersistida["Base HANA<br/>Persistida"]
    AppCAP["App CAP"]
    APO["APO"]
    GestaoPerdas["Gestão de perdas (CAP)"]
    Job["Job"]

    O9Bucket -->|1| CAPLeituraS3
    CAPLeituraS3 -->|2| BaseHANAPersistida
    BaseHANAPersistida -->|Material + Centro| AppCAP
    AppCAP --> GestaoPerdas
    APO --> GestaoPerdas
    GestaoPerdas -->|Notifica| BaseHANAPersistida
    GestaoPerdas -->|44.3| Job
