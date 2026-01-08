### Descrição

Este repositório contém a implementação técnica para habilitar as funções de **Espionagem (Spy)**, **Sussurro (Whisper)** e **Intercalação (Barge)** em sistemas FreePBX 16/IncrediblePBX baseados exclusivamente em tecnologia **PJSIP**.

Diferente do recurso padrão `555`, esta solução foca em:

* **Segurança Robusta:** Exige um código PIN de supervisor via função `Authenticate` antes de liberar o áudio.
* **Discagem Direta:** Permite monitorar um ramal específico imediatamente através de um prefixo customizável (`222` + número do ramal).
* **Modo Whisper Nativo:** A chamada inicia automaticamente no modo de treinamento (Sussurro), onde o supervisor fala com o agente sem que o cliente ouça.

---

### Conteúdo Detalhado

Abaixo, está a tabela de comandos DTMF para o controle da chamada:

| Comando DTMF | Ação Resultante | Comportamento do Áudio |
| --- | --- | --- |
| **Pressionar 4** | **Modo Whisper** | O supervisor fala e apenas o agente interno ouve. |
| **Pressionar 5** | **Modo Barge** | Transforma a chamada em conferência (todos se ouvem). |
| **Pressionar 6** | **Modo Listen** | O supervisor apenas ouve (mudo para ambos os lados). |
| **Pressionar *** | **Next Channel** | Pula para a próxima chamada ativa se o alvo for um grupo. |

---

### Guia de Instalação Rápida

1. Acesse a página de configurações customizadas: `Admin, Config Edit`.
2. Edite o arquivo `extensions_custom.conf` e insira o bloco de código fornecido.
   ```
   [from-internal-custom]
   ; --- INÍCIO DO MÓDULO DE MONITORAMENTO ---
   ; Descrição: Permite espiar (Spy) e sussurrar (Whisper) em ramais PJSIP
   ; Gatilho: 222 + Número do Ramal (Ex: 2222000)

   exten => _222.,1,NoOp(Iniciando Monitoramento: Supervisor ${CALLERID(num)} -> Alvo ${EXTEN:3})
   ; 1. Autenticação: Exige PIN para evitar acesso não autorizado. O PIN pode e deve ser alterado.
    same => n,Authenticate(1234)
 
   ; 2. Feedback Sonoro: Avisa o supervisor para aguardar a conexão
    same => n,Playback(pls-wait-connect-call)
   
   ; 3. Execução do ChanSpy:
   ; Flag 'd': Permite alternar modos usando o teclado (4, 5 ou 6)
   ; Flag 'w': Inicia automaticamente no modo Whisper (Sussurro)
   ; ${EXTEN:3}: Remove o prefixo '222' e isola o número do ramal alvo
     same => n,ChanSpy(PJSIP/${EXTEN:3},dw)
 
   ; 4. Finalização
     same => n,Hangup()
   ; --- FIM DO MÓDULO ---

   ```
4. **Atenção ao Offset:** O código utiliza `${EXTEN:3}`. Isso significa que se o prefixo for `222`, os 3 primeiros dígitos serão ignorados para identificar o ramal alvo.
5. Aplique as configurações clicando em `Submit` e depois em `Apply`.

---
