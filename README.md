# Game-Elfo-Guerreiro
Game Elfo

import pygame
import random
import os

# Diretório de recursos
diretorio_atual = os.path.dirname(__file__)

# Inicialização do Pygame
pygame.init()

# Cores
branco = (255, 255, 255)
preto = (0, 0, 0)
pedramusgo = (128, 128, 128)
vermelho = (255, 0, 0)
marrom = (139, 69, 19)  # Cor para as montanhas

# Fator de zoom inicial
zoom = 1

# Configurações da tela
largura_tela = 1115
altura_tela = 610
tamanho_bloco = 80
tamanho_bloco2 = 1000
velocidade_jogo = 50

# Configuração da fonte
fonte_jogo = pygame.font.SysFont(None, 25)

# Configuração da tela
tela = pygame.display.set_mode((largura_tela, altura_tela))
pygame.display.set_caption('Jogo do Mario')

# Carregamento das imagens (com caminho completo)
imagem_fundo = pygame.image.load(os.path.join(diretorio_atual, 'background_mario.png')).convert()
imagem_mario_direita = pygame.image.load(os.path.join(diretorio_atual, 'mario_direita.png')).convert_alpha()
imagem_mario_esquerda = pygame.transform.flip(imagem_mario_direita, True, False)
imagem_cogumelo = pygame.image.load(os.path.join(diretorio_atual, 'cogumelo.png')).convert_alpha()
imagem_obstaculo = pygame.image.load(os.path.join(diretorio_atual, 'obstaculo.png')).convert_alpha()
imagem_inimigo = pygame.image.load(os.path.join(diretorio_atual, 'inimigo.png')).convert_alpha()

# Redimensionamento das imagens
imagem_fundo = pygame.transform.scale(imagem_fundo, (largura_tela, altura_tela))
imagem_mario_direita = pygame.transform.scale(imagem_mario_direita, (tamanho_bloco, tamanho_bloco))
imagem_mario_esquerda = pygame.transform.scale(imagem_mario_esquerda, (tamanho_bloco, tamanho_bloco))
imagem_cogumelo = pygame.transform.scale(imagem_cogumelo, (tamanho_bloco2, tamanho_bloco2))
imagem_obstaculo = pygame.transform.scale(imagem_obstaculo, (tamanho_bloco, tamanho_bloco))
imagem_inimigo = pygame.transform.scale(imagem_inimigo, (tamanho_bloco, tamanho_bloco))

# Parâmetros físicos do jogo
gravidade = 0.1  # Gravidade global do jogo
velocidade_pulo = -6  # Ajustando a velocidade do pulo

# Função para exibir a pontuação na tela
def exibir_pontuacao(pontuacao):
    """Exibe a pontuação do jogador na tela."""
    texto = fonte_jogo.render("Pontuação: " + str(pontuacao), True, branco)
    tela.blit(texto, [0, 0])

# Função para exibir a vida do Mario
def exibir_vida(vida):
    """Exibe a vida do Mario na tela."""
    texto = fonte_jogo.render("Vidas: " + str(vida), True, branco)
    tela.blit(texto, [largura_tela - 100, 0])

# Função para desenhar o Mario na tela
def desenhar_mario(x, y, direcao):
    """Desenha o Mario na tela, na posição (x, y) e na direção especificada."""
    if direcao == "direita":
        tela.blit(imagem_mario_direita, (x, y))
    elif direcao == "esquerda":
        tela.blit(imagem_mario_esquerda, (x, y))

# Classe para representar os obstáculos
class Obstaculo:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        self.imagem = imagem_obstaculo
        self.rect = self.imagem.get_rect(topleft=(self.x, self.y))

    def desenhar(self):
        tela.blit(self.imagem, (self.x, self.y))

# Classe para representar os inimigos
class Inimigo:
    def __init__(self, x, y, plataforma):
        self.x = x
        self.y = y
        self.imagem = imagem_inimigo
        self.rect = self.imagem.get_rect(topleft=(self.x, self.y))
        self.direcao = "esquerda"
        self.velocidade = 2
        self.vida = 1
        self.plataforma = plataforma  # Plataforma onde o inimigo se move

    def desenhar(self):
        tela.blit(self.imagem, (self.x, self.y))

    def mover(self, mario_x):
        if abs(self.x - mario_x) < 300:  # Se o Mario estiver a menos de 300 pixels de distância
            if self.x < mario_x:
                self.direcao = "direita"
            else:
                self.direcao = "esquerda"

            if self.direcao == "esquerda":
                self.x -= self.velocidade
            elif self.direcao == "direita":
                self.x += self.velocidade
        else:
            # Patrulha a plataforma
            if self.direcao == "esquerda":
                self.x -= self.velocidade
            elif self.direcao == "direita":
                self.x += self.velocidade

            # Inverte a direção se o inimigo atingir as bordas da plataforma
            if self.x <= self.plataforma.x or self.x >= self.plataforma.x + self.plataforma.largura - tamanho_bloco:
                self.direcao = "direita" if self.direcao == "esquerda" else "esquerda"

    def verificar_colisao_ataque(self, mario_rect, atacando):
        if atacando and self.rect.colliderect(mario_rect):
            self.vida -= 1
            return True
        return False

# Classe para representar as plataformas
class Plataforma:
    def __init__(self, x, y, largura, altura):
        self.x = x
        self.y = y
        self.largura = largura
        self.altura = altura
        self.rect = pygame.Rect(self.x, self.y, self.largura, self.altura)

    def desenhar(self):
        pygame.draw.rect(tela, pedramusgo, self.rect)  # Desenha a plataforma como um retângulo

# Função para gerar plataformas aleatórias
def gerar_plataformas(altura_maxima):
    plataformas = []
    altura_atual = altura_tela
    while altura_atual > altura_maxima:
        largura_plataforma = random.randint(50, 200)
        x = random.randint(0, largura_tela - largura_plataforma)
        altura_atual -= random.randint(50, 100)  # Ajustar a altura de cada plataforma
        plataformas.append(Plataforma(x, altura_atual, largura_plataforma, 30))
    return plataformas

def colidir_com_plataformas(x_mario, y_mario, x_velocidade, y_velocidade, plataformas, pulando):
    """Trata a colisão do Mario com as plataformas."""
    mario_rect = pygame.Rect(x_mario, y_mario, tamanho_bloco, tamanho_bloco)
    for plataforma in plataformas:
        if plataforma.rect.colliderect(mario_rect):
            # Verifica se a colisão seria por cima
            if y_mario + tamanho_bloco >= plataforma.rect.top and y_velocidade > 0:
                y_mario = plataforma.rect.top - tamanho_bloco
                y_velocidade = 0
                return x_mario, y_mario, False  # Mario não está pulando

            # Verifica se a colisão seria por baixo
            elif y_mario < plataforma.rect.bottom and y_velocidade < 0:
                y_mario = plataforma.rect.bottom
                y_velocidade = 0
                return y_mario, y_velocidade, False  # Mario está pulando

            # Caso contrário, trata colisões laterais
            else:
                # Verifica a direção do movimento para reposicionar corretamente
                if x_velocidade > 0:  # Indo para a direita
                    x_mario = plataforma.rect.left - tamanho_bloco
                elif x_velocidade < 0:  # Indo para a esquerda
                    x_mario = plataforma.rect.right
                return x_mario, y_mario, pulando  # Mantém o estado de pulo

    return x_mario, y_mario, pulando  # Retorna os valores originais se não houver colisão

# Função principal do jogo
def jogo():
    """Contém o loop principal do jogo e gerencia a lógica do jogo."""
    fim_jogo = False
    jogo_encerrado = False

    # Posição inicial do Mario
    x_mario = 100  # Posição inicial à esquerda
    y_mario = altura_tela - tamanho_bloco

    # Velocidade horizontal e vertical do Mario
    x_velocidade = 0
    y_velocidade = 0

    # Estado de pulo do Mario
    pulando = False

    # Pontuação inicial do jogador
    pontuacao = 0

    # Direção inicial do Mario
    direcao = "direita"

    # Relógio para controlar o tempo do jogo
    relogio = pygame.time.Clock()

    # Criação de plataformas usando a função gerar_plataformas
    plataformas = gerar_plataformas(100)  # Ajustar a altura máxima conforme necessário

    # Criação de inimigos (associado à plataforma) - Você pode adicionar lógica para gerar inimigos em plataformas aleatórias
    inimigos = []
    for plataforma in plataformas:
        if random.random() < 0.2:  # 20% de chance de gerar um inimigo na plataforma
            inimigo_x = random.randint(plataforma.x, plataforma.x + plataforma.largura - tamanho_bloco)
            inimigo_y = plataforma.y - tamanho_bloco
            inimigos.append(Inimigo(inimigo_x, inimigo_y, plataforma))

    # Variável para controlar o ataque do Mario
    atacando = False
    tempo_ataque = 0

    # Vida do Mario
    vida_mario = 5

    # Variáveis para controle da imagem de fundo e câmera
    fundo_x = 0
    fundo_y = 0  # Posição vertical inicial do fundo
    velocidade_fundo = 2
    fator_parallax = 0.5  # Fator de ajuste para o movimento vertical do fundo
    camera_y = 0  # Posição vertical da câmera

    # Loop principal do jogo
    while not fim_jogo:
        # Loop da tela de fim de jogo
        while jogo_encerrado is True:
            tela.fill(preto)
            mensagem = fonte_jogo.render("Você perdeu! Pressione C-Jogar Novamente ou Q-Sair", True, vermelho)
            tela.blit(mensagem, [largura_tela // 6, altura_tela // 3])  # // para divisão inteira
            pygame.display.update()

            # Verificação de eventos na tela de fim de jogo
            for event in pygame.event.get():
                if event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_q:
                        fim_jogo = True
                        jogo_encerrado = False
                    if event.key == pygame.K_c:
                        jogo()

        # Verificação de eventos durante o jogo
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                fim_jogo = True
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT:
                    x_velocidade = -5
                    direcao = "esquerda"
                elif event.key == pygame.K_RIGHT:
                    x_velocidade = 5
                    direcao = "direita"
                elif event.key == pygame.K_UP and not pulando:
                    pulando = True
                    y_velocidade = velocidade_pulo
                elif event.key == pygame.K_SPACE:
                    atacando = True
                    tempo_ataque = 10  # Define a duração do ataque em frames
            if event.type == pygame.KEYUP:
                if event.key == pygame.K_LEFT or event.key == pygame.K_RIGHT:
                    x_velocidade = 0
                elif event.key == pygame.K_SPACE:
                    atacando = False

        # Aplica o ataque
        if atacando:
            tempo_ataque -= 1
            if tempo_ataque <= 0:
                atacando = False

        # Aplica o pulo:
        if pulando:
            y_mario += y_velocidade  # Aplica a velocidade do pulo à posição y
            y_velocidade += gravidade  # Aplica a gravidade à velocidade do pulo
            if y_velocidade >= 0:  # Verifica se o Mario atingiu o pico do pulo
                pulando = False  # Define pulando como False para interromper o pulo
                y_velocidade = 0  # Reseta a velocidade vertical

        # Aplica a gravidade ao Mario APENAS se ele NÃO estiver colidindo com uma plataforma
        else:
            y_mario += y_velocidade
            y_velocidade += gravidade

        # Verificação de colisão com plataformas
        x_mario, y_mario, pulando = colidir_com_plataformas(x_mario, y_mario, x_velocidade, y_velocidade, plataformas,
                                                            pulando)  # Passa 'pulando' como argumento plataformas)

        # Atualização da posição do Mario (após verificar colisões)
        x_mario += x_velocidade

        # Verificação se o Mario caiu da tela
        if y_mario > altura_tela:
            vida_mario -= 1  # Mario perde uma vida ao cair
            if vida_mario <= 0:  # Fim do jogo se a vida chegar a zero
                jogo_encerrado = True
            else:
                # Reposiciona o Mario no início
                x_mario = 100
                y_mario = altura_tela - tamanho_bloco
                x_velocidade = 0
                y_velocidade = 0

        # Manutenção do Mario dentro dos limites da tela
        if x_mario >= largura_tela:
            x_mario = largura_tela - tamanho_bloco
        if x_mario < 0:
            x_mario = 0

        # --- Movimento da câmera ---
        camera_y = max(0, y_mario - altura_tela // 2)  # Ajusta a posição da câmera

        # --- Movimento do fundo ---
        movimento_fundo_y = camera_y * fator_parallax  # Calcula o movimento do fundo

        # Calcula o novo fator de zoom com base na altura do Mario
        zoom = max(1, 1 + (altura_tela // 2 - (y_mario - camera_y)) / 500)  # Ajusta o divisor para controlar a velocidade do zoom

        # Redimensiona a imagem do fundo dinamicamente
        imagem_fundo_redimensionada = pygame.transform.scale(
            imagem_fundo, (int(largura_tela * zoom), int(altura_tela * zoom))
        )

        # Calcula a posição x do fundo para centralizá-lo
        fundo_x_zoom = fundo_x * zoom

        # Desenha o fundo com zoom e parallax
        tela.blit(imagem_fundo_redimensionada, (fundo_x_zoom, fundo_y - movimento_fundo_y))
        tela.blit(imagem_fundo_redimensionada, (fundo_x_zoom + imagem_fundo_redimensionada.get_width(), fundo_y - movimento_fundo_y))

        # Move o fundo na horizontal
        if x_velocidade > 0:  # Mario indo para a direita
            fundo_x -= velocidade_fundo
        elif x_velocidade < 0:  # Mario indo para a esquerda
            fundo_x += velocidade_fundo

        # Repete o fundo na horizontal
        if fundo_x <= -largura_tela:
            fundo_x = 0
        elif fundo_x >= largura_tela:
            fundo_x = -largura_tela

        # Desenho das plataformas
        for plataforma in plataformas:
            plataforma.desenhar()

        # Desenho dos inimigos
        for inimigo in inimigos:
            inimigo.desenhar()
            inimigo.mover(x_mario)  # Passa a posição x do Mario

            # Verificação de colisão com ataque do Mario
            if inimigo.verificar_colisao_ataque(pygame.Rect(x_mario, y_mario, tamanho_bloco, tamanho_bloco), atacando):
                if inimigo.vida <= 0:
                    inimigos.remove(inimigo)
                    pontuacao += 10

            # Verificação de colisão com o inimigo
            if inimigo.rect.colliderect(pygame.Rect(x_mario, y_mario, tamanho_bloco, tamanho_bloco)):
                vida_mario -= 1  # Mario perde uma vida ao colidir com o inimigo
                if vida_mario <= 0:  # Fim do jogo se a vida chegar a zero
                    jogo_encerrado = True

        # Desenho do Mario
        desenhar_mario(x_mario, y_mario, direcao)

        # Exibição da pontuação
        exibir_pontuacao(pontuacao)

        # Exibir vida do Mario
        exibir_vida(vida_mario)

        # Atualização da tela
        pygame.display.update()

        # Controle da velocidade do jogo
        relogio.tick(velocidade_jogo)

    # Encerramento do Pygame
    pygame.quit()
    quit()

# Início do jogo
jogo()
