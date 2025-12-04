Projeto de Failover Automático com IP-SLA (Linux + Ansible)

Este projeto implementa failover automático de rotas em servidores Linux utilizando IP-SLA personalizado, tabelas de roteamento, Ansible e regras de política (ip rule).
Ele permite alternar automaticamente entre gateway primário e gateway de backup, baseado em testes de conectividade programados.

Funcionalidades Principais

Failover automático entre rota primária e rota de backup

Reset, ativação e desativação do failover via Ansible

Instalação e desinstalação completa do agente IP-SLA

Forçar manualmente rota primária ou backup

Geração de relatório HTML com status das rotas

Sistema modular baseado em variáveis em group_vars

📦 Instalação & Uso
▶️ Instalar o agente IP-SLA
ansible-playbook playbooks/deploy_ip_sla_timer.yml -l 177.53.16.9

🔄 Forçar rota primária
ansible-playbook playbooks/force_primary.yml -l 177.53.16.9

🔄 Forçar rota de backup
ansible-playbook playbooks/force_backup.yml

🧹 Limpar tabelas de roteamento
ansible-playbook playbooks/clear_table.yml -l 177.53.16.9

📊 Mostrar tabela e gerar relatório HTML
ansible-playbook playbooks/show_rt2.yml -l 177.53.16.9

❌ Desinstalar todo o script
ansible-playbook playbooks/uninstall_ip_sla.yaml -l 177.53.16.9

🔁 Resetar configurações
ansible-playbook -i inventory/hosts.yml playbooks/reset_ip_sla.yml -l 177.53.16.9

Reset + subir rota de backup automaticamente
ansible-playbook -i inventory/hosts.yml playbooks/reset_ip_sla.yml -e prime_mode=backup -l 177.53.16.9

Reset + deixar failover decidir rota
ansible-playbook playbooks/reset_ip_sla.yml -e prime_mode=none -l 177.53.16.9

📴 Desabilitar failover (sem desinstalar)
ansible-playbook playbooks/deactivate_failover.yml -l 177.53.16.9


Para reabilitar, execute novamente o playbook reset_ip_sla.yml.

⚙️ Estrutura de Variáveis

As variáveis ficam em group_vars, divididas por grupos de hosts.

## 🌍 Variáveis globais — group_vars/all.yml
ip_sla:
  table_name: "rt2"
  table_id: 200
  policy_rule_priority: 1000
  gw_primary: "10.10.10.1"
  gw_backup: "10.10.10.2"
  tracked_dests:
    - 10.11.12.6
    - 10.11.10.44
    - 10.21.2.6
    - 10.21.3.24
    - 10.41.2.6
    - 10.41.2.8
    - 10.11.11.42
    - 10.21.3.25
    - 10.11.8.124
    - 10.11.9.150
  min_ok: 1
  subnet_cidr: "10.10.10.0/24"
  timer_every: "30s"

🏬 Variáveis específicas por grupo — group_vars/site_padrao.yml
ip_sla:
  gw_primary: "10.45.0.70"
  gw_backup: "10.45.10.68"
  tracked_dests:
    - 10.11.10.44
    - 10.11.12.6
    - 10.41.2.6

🔎 Precedência das Variáveis

A ordem de aplicação é:

all.yml → aplica para todos os hosts

Arquivo do grupo (ex: site_padrao.yml) → sobrescreve variáveis globais

Sempre que houver conflito, o arquivo mais específico vence.

🛠️ Variáveis Disponíveis
Variável	Função
table_name	Nome da tabela de roteamento
table_id	ID numérico da tabela
policy_rule_priority	Prioridade da regra ip rule
gw_primary	Gateway primário
gw_backup	Gateway de backup
tracked_dests	Destinos monitorados via ping
min_ok	Mínimo de destinos acessíveis para manter rota primária
subnet_cidr	Sub-rede utilizada nas rotas
timer_every	Intervalo entre testes
ping_timeout	Timeout do ping
failover_trigger	Nº de falhas consecutivas para acionar failover
🧭 Boas Práticas

Manter variáveis globais em all.yml

Utilizar arquivos específicos de grupo para sobrescrever detalhes locais

Evitar duplicidade entre arquivos

Após ajustes importantes, recomenda-se:

🔄 Desinstalar:
ansible-playbook playbooks/uninstall_ip_sla.yaml -l <host>

▶️ Reinstalar:
ansible-playbook playbooks/deploy_ip_sla_timer.yml -l <host>



