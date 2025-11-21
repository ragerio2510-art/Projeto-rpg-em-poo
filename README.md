from abc import ABC, abstractmethod
import random

class Dado:
    @staticmethod
    def rolar(lados=6):
        return random.randint(1, lados)

class Combatente:
    def __init__(self, nome, vida, forca):
        self.nome = nome
        self._vida = vida
        self.vida_max = vida
        self.forca = forca

    @property
    def vida(self):
        return self._vida

    @vida.setter
    def vida(self, valor):
        # Protege a vida para que nunca seja negativa e não ultrapasse o máximo
        if valor < 0:
            self._vida = 0
        elif valor > self.vida_max:
            self._vida = self.vida_max
        else:
            self._vida = valor

    def receber_dano(self, dano):
        # Garante que vida não fique abaixo de zero
        self.vida = self.vida - dano
        print(f"{self.nome} recebeu {dano} de dano. Vida agora: {self.vida}/{self.vida_max}")

    def esta_vivo(self):
        return self.vida > 0

    def atacar(self, alvo):
        # Método padrão (pode ser sobrescrito)
        dano_variacao = Dado.rolar(6) - 1  # 0..5 extra
        dano = self.forca + dano_variacao
        print(f"{self.nome} ataca {alvo.nome} com dano base {self.forca} + var {dano_variacao} = {dano}")
        alvo.receber_dano(dano)
        return dano

    def __str__(self):
        return f"{self.nome} | Vida: {self.vida}/{self.vida_max} | Força: {self.forca}"


class Arma:
    def __init__(self, nome, dano):
        self.nome = nome
        self.dano = dano

    def __str__(self):
        return f"{self.nome} (Dano: {self.dano})"

class Pocao:
    def __init__(self, nome, cura):
        self.nome = nome
        self.cura = cura

    def __str__(self):
        return f"{self.nome} (Cura: {self.cura})"

espada_longa = Arma("Espada Longa", 10)
cajado_magico = Arma("Cajado Mágico", 8)
pocao_vida = Pocao("Poção de Vida", 20)

class Inventario:
    def __init__(self):
        self.itens = []

    def adicionar_item(self, item):
        self.itens.append(item)
        print(f"Item {item} adicionado ao inventário.")

    def usar_pocao(self, personagem):
        for i, item in enumerate(self.itens):
            if isinstance(item, Pocao):
                personagem.vida = personagem.vida + item.cura
                print(f"{personagem.nome} usou {item.nome} e recuperou {item.cura} pontos de vida.")
                del self.itens[i]
                return True
        print("Nenhuma poção disponível no inventário.")
        return False

    def __str__(self):
        if not self.itens:
            return "Inventário vazio"
        return "Inventário: " + ", ".join(str(i) for i in self.itens)


class Personagem(Combatente):
    def __init__(self, nome, vida, forca):
        super().__init__(nome, vida, forca)
        self.inventario = Inventario()
        self.arma = None
        self.habilidades = []

    def equipar_arma(self, arma):
        self.arma = arma
        print(f"{self.nome} equipou {arma.nome}.")

    def atacar(self, alvo):
        # Atacar considera arma equipada + dado para variação
        dano_base = self.forca + (self.arma.dano if self.arma else 0)
        dado = Dado.rolar(6) - 1  # 0..5
        dano = dano_base + dado
        print(f"{self.nome} ataca {alvo.nome}: forca {self.forca} + arma {(self.arma.dano if self.arma else 0)} + var {dado} = {dano}")
        alvo.receber_dano(dano)
        return dano

    def usar_habilidade(self, nome_hab, alvo):
        for hab in self.habilidades:
            if hab.__class__.__name__ == nome_hab or getattr(hab, "nome", "") == nome_hab:
                hab.usar(self, alvo)
                return True
        print(f"{self.nome} não possui a habilidade {nome_hab}.")
        return False

class Guerreiro(Personagem):
    def __init__(self, nome, vida=100, forca=12):
        super().__init__(nome, vida, forca)

class Mago(Personagem):
    def __init__(self, nome, vida=70, forca=6, magia=12):
        super().__init__(nome, vida, forca)
        self.magia = magia

    def atacar(self, alvo):
        # Mago usa magia em vez de arma
        dano_base = self.forca + self.magia
        dado = Dado.rolar(6) - 1
        dano = dano_base + dado
        print(f"{self.nome} conjura magia em {alvo.nome}: forca {self.forca} + magia {self.magia} + var {dado} = {dano}")
        alvo.receber_dano(dano)
        return dano

class Arqueiro(Personagem):
    def __init__(self, nome, vida=80, forca=8, precisao=10):
        super().__init__(nome, vida, forca)
        self.precisao = precisao

    def atacar(self, alvo):
        # Arqueiro conta com precisão
        dano_base = self.forca + self.precisao + (self.arma.dano if self.arma else 0)
        dado = Dado.rolar(6) - 1
        dano = dano_base + dado
        print(f"{self.nome} dispara em {alvo.nome}: forca {self.forca} + precisao {self.precisao} + arma {(self.arma.dano if self.arma else 0)} + var {dado} = {dano}")
        alvo.receber_dano(dano)
        return dano


class Monstro(Combatente):
    def __init__(self, nome, vida=30, dano=5):
        super().__init__(nome, vida, dano)

    def atacar(self, alvo):
        dado = Dado.rolar(6) - 1
        dano = self.forca + dado
        print(f"{self.nome} ataca {alvo.nome} com dano {self.forca} + var {dado} = {dano}")
        alvo.receber_dano(dano)
        return dano


class MonstroFactory:
    @staticmethod
    def criar_goblin():
        return Monstro("Goblin", vida=30, dano=5)

class Orc(Monstro):
    def __init__(self, nome="Orc", vida=50, dano=8):
        super().__init__(nome, vida, dano)

    def atacar(self, alvo):
        critico = random.random() < 0.2
        dado = Dado.rolar(6) - 1
        dano = (self.forca * 2 if critico else self.forca) + dado
        print(f"{self.nome} ataca {alvo.nome} {'(CRÍTICO) ' if critico else ''}com dano = {dano}")
        alvo.receber_dano(dano)
        return dano

class Habilidade(ABC):
    @abstractmethod
    def usar(self, usuario, alvo):
        pass

class AtaqueForte(Habilidade):
    def usar(self, usuario, alvo):
        dano = usuario.forca * 2
        print(f"{usuario.nome} usa AtaqueForte em {alvo.nome} causando {dano} de dano!")
        alvo.receber_dano(dano)

class BolaDeFogo(Habilidade):
    def usar(self, usuario, alvo):
        dano = usuario.forca + 15
        print(f"{usuario.nome} usa BolaDeFogo em {alvo.nome} causando {dano} de dano!")
        alvo.receber_dano(dano)

class Batalha:
    def __init__(self, c1, c2):
        self.c1 = c1
        self.c2 = c2

    def iniciar(self):
        print(f"Iniciando batalha entre {self.c1.nome} e {self.c2.nome}")
        turno = 0
        while self.c1.esta_vivo() and self.c2.esta_vivo():
            turno += 1
            print(f"\n--- Turno {turno} ---")
            # c1 ataca primeiro no turno ímpar
            if turno % 2 == 1:
                if self.c1.esta_vivo():
                    self.c1.atacar(self.c2)
                if self.c2.esta_vivo():
                    self.c2.atacar(self.c1)
            else:
                if self.c2.esta_vivo():
                    self.c2.atacar(self.c1)
                if self.c1.esta_vivo():
                    self.c1.atacar(self.c2)

        vencedor = self.c1 if self.c1.esta_vivo() else self.c2
        print(f"\nBatalha encerrada. Vencedor: {vencedor.nome}")
        return vencedor

class BatalhaEquipe:
    def __init__(self, equipe1, equipe2):
        self.e1 = equipe1
        self.e2 = equipe2

    def iniciar(self):
        print("Iniciando batalha entre equipes.")
        rodada = 0
        while any(p.esta_vivo() for p in self.e1) and any(p.esta_vivo() for p in self.e2):
            rodada += 1
            print(f"\n=== Rodada de equipe {rodada} ===")
            # cada membro da equipe enfrenta o membro correspondente (zip)
            for p1, p2 in zip(self.e1, self.e2):
                if p1.esta_vivo() and p2.esta_vivo():
                    p1.atacar(p2)
                if p2.esta_vivo() and p1.esta_vivo():
                    p2.atacar(p1)
            # se tamanhos diferentes, membros sobrando atacam aleatoriamente
            if len(self.e1) > len(self.e2):
                for sobra in self.e1[len(self.e2):]:
                    if sobra.esta_vivo():
                        alvo = random.choice([x for x in self.e2 if x.esta_vivo()]) if any(x.esta_vivo() for x in self.e2) else None
                        if alvo:
                            sobra.atacar(alvo)
            elif len(self.e2) > len(self.e1):
                for sobra in self.e2[len(self.e1):]:
                    if sobra.esta_vivo():
                        alvo = random.choice([x for x in self.e1 if x.esta_vivo()]) if any(x.esta_vivo() for x in self.e1) else None
                        if alvo:
                            sobra.atacar(alvo)

        vivos1 = any(p.esta_vivo() for p in self.e1)
        vencedor = "Equipe 1" if vivos1 else "Equipe 2"
        print(f"\nBatalha de equipes encerrada. Vencedora: {vencedor}")
        return vencedor

if __name__ == "__main__":
    # Criação de personagens (questões 1-3)
    g = Guerreiro("Arthos")
    m = Mago("Merlin")
    a = Arqueiro("Silvana")

    # Equipar armas (questões 7 e 9)
    g.equipar_arma(espada_longa)
    m.equipar_arma(cajado_magico)  # embora mago use magia, ok ter arma
    a.equipar_arma(Arma("Arco Curto", 6))

    # Inventário e poção (questões 8, 15, 16, 17)
    g.inventario.adicionar_item(pocao_vida)
    m.inventario.adicionar_item(Pocao("Poção Pequena", 10))

    # Criar monstros (questões 4, 19)
    gob1 = MonstroFactory.criar_goblin()
    orc1 = Orc()

    # Adicionar habilidades (questões 21-23)
    g.habilidades.append(AtaqueForte())
    m.habilidades.append(BolaDeFogo())
    a.habilidades.append(AtaqueForte())  # arqueiro também pode ter

    # Pequena batalha 1 (questões 14, 25, 27-29): Guerreiro vs Goblin
    print("\n--- Batalha: Guerreiro vs Goblin ---")
    batalha1 = Batalha(g, gob1)
    vencedor1 = batalha1.iniciar()

    # Usar poção se ferido (questão 17)
    if g.esta_vivo() and g.vida < g.vida_max:
        g.inventario.usar_pocao(g)

    # Batalha de equipe (questão 30): equipe heróis vs orcs
    print("\n--- Batalha em Equipe: Heróis vs Orcs ---")
    herois = [g, m, a]
    orcs = [Orc("Orc1"), Orc("Orc2"), Orc("Orc3")]
    batalha_e = BatalhaEquipe(herois, orcs)
    vencedor_equipe = batalha_e.iniciar()

    # Exemplo de usar habilidade (questão 23)
    if m.esta_vivo() and orc1.esta_vivo():
        m.usar_habilidade("BolaDeFogo", orc1)
