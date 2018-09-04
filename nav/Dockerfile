FROM microsoft/dynamics-nav

HEALTHCHECK --interval=10s --timeout=8s --start-period=300s --retries=3 CMD [ "powershell", ".\\Run\\HealthCheck.ps1" ]