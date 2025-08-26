# Criando e Depurando uma Topologia em Árvore no Mininet

Este documento detalha o processo de criação de uma topologia de rede em árvore com o Mininet. Ele aborda os erros comuns relacionados à ausência de controladores SDN e apresenta a solução mais robusta para executar a simulação.

## A Topologia em Árvore

O objetivo é criar uma rede com a seguinte estrutura:
* **1 Switch Core:** `s1`
* **2 Switches de Agregação/Borda:** `s2` e `s3`, cada um conectado ao switch core.
* **6 Hosts:**
    * `h1`, `h2`, `h3` conectados ao switch `s2`.
    * `h4`, `h5`, `h6` conectados ao switch `s3`.

### Diagrama da Topologia

Abaixo está uma representação visual da rede, conforme solicitado.

![Diagrama da Topologia Tree](diagrama_tree.png)

*(Nota: Este código assume que existe uma imagem chamada `diagrama_tree.png` no mesmo diretório. Uma forma de gerar este diagrama é descrita no final deste documento.)*

## O Desafio: Erros de Controlador

Ao tentar executar a topologia, é comum encontrar erros se o ambiente não estiver perfeitamente configurado. O script tenta, por padrão, invocar um processo de controlador para gerenciar o fluxo de pacotes nos switches.

### Erro 1: "Cannot find required executable controller"

Este é o primeiro erro encontrado. Ele ocorre quando o Mininet tenta usar sua classe `Controller` padrão, que depende de um controlador simples (geralmente do framework POX) que não está instalado ou não se encontra no `$PATH` do sistema.

### Erro 2: "Cannot find required executable ovs-controller"

Ao tentar corrigir o primeiro erro usando a classe `OVSController`, um segundo erro similar pode aparecer. Desta vez, o Mininet procura pelo executável `ovs-controller`, uma ferramenta que acompanha o Open vSwitch. Se a instalação do Open vSwitch for mínima ou estiver incompleta, este executável pode estar ausente.

## A Solução Definitiva: Modo Standalone

A abordagem mais simples, robusta e portável para topologias simples é configurar o Mininet para rodar **sem um processo de controlador explícito**.

Nesse modo, conhecido como **standalone**, os switches Open vSwitch (`OVSKernelSwitch`) operam de forma autônoma. Por padrão, eles se comportam como switches L2 de aprendizado (MAC learning switches), que é exatamente o necessário para que os hosts na rede possam se comunicar.

A implementação é feita garantindo duas coisas no código:
1.  A rede é instanciada com o parâmetro `controller=None`.
2.  Nenhum controlador é adicionado posteriormente com `net.addController()`.

## Código Final e Completo

Este é o código final, corrigido para rodar em modo standalone, eliminando a dependência de executáveis externos.

```python
from mininet.topo import Topo
from mininet.net import Mininet
from mininet.node import OVSKernelSwitch
from mininet.cli import CLI
from mininet.log import setLogLevel

class TreeTopo(Topo):
    def build(self):
        core = self.addSwitch('s1')

        s2 = self.addSwitch('s2')
        s3 = self.addSwitch('s3')

        self.addLink(core, s2)
        self.addLink(core, s3)

        for i in range(1, 4):
            h = self.addHost(f'h{i}')
            self.addLink(s2, h)

        for i in range(4, 7):
            h = self.addHost(f'h{i}')
            self.addLink(s3, h)

def run():
    topo = TreeTopo()
    net = Mininet(
        topo=topo,
        switch=OVSKernelSwitch,
        controller=None
    )
    net.start()
    print("Rede iniciada. Use 'pingall' para testar conectividade.")
    CLI(net)
    net.stop()

if __name__ == '__main__':
    setLogLevel('info')
    run()
