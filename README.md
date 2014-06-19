# Four-In-A-Row (a Connect Four clone)
# By Al Sweigart al@inventwithpython.com
# http://inventwithpython.com/pygame
# Released under a "Simplified BSD" license

import random, copy, sys, pygame
from pygame.locals import *

範圍=range 
邊界寬 = 7  # how many spaces wide the 版子 is
邊界高 = 6 # how many spaces tall the 版子 is
assert 邊界寬 >= 4 and 邊界高 >= 4, 'Board must be at least 4x4.'

難度 = 2 # how many moves to look ahead. (>2 is usually too much)

空間大小 = 50 # size of the tokens and individual 版子 spaces in pixels

架框更新頻率 = 30 # frames per second to update the screen
畫面寬 = 640 # width of the program's window, in pixels
畫面高 = 480 # height in pixels

X軸 = int((畫面寬 - 邊界寬 * 空間大小) / 2)
Y軸 = int((畫面高 - 邊界高 * 空間大小) / 2)

亮藍 = (0, 50, 255)
白色 = (255, 255, 255)

背景顏色 = 亮藍
格式顏色 = 白色

紅棋 = '紅'
黑棋 = '黑'
空處 = None
玩家 = '玩家'
電腦 = '電腦'


def 主函式():
    global 頻率時間, 展示畫面, 紅色空間, 黑色空間, 紅色硬幣
    global 黑色硬幣路徑, 版子畫面, 箭頭畫面, 箭頭方向, 玩家獲勝畫面
    global 電腦獲勝畫面, 贏家框, 平手畫面

    pygame.init()
    頻率時間 = pygame.time.Clock()
    展示畫面 = pygame.display.set_mode((畫面寬, 畫面高))
    pygame.display.set_caption('Four in a Row')

    紅色空間 = pygame.Rect(int(空間大小 / 2), 畫面高 - int(3 * 空間大小 / 2), 空間大小, 空間大小)
    黑色空間 = pygame.Rect(畫面寬 - int(3 * 空間大小 / 2), 畫面高 - int(3 * 空間大小 / 2), 空間大小, 空間大小)
    紅色硬幣 = pygame.image.load('4row_red.png')
    紅色硬幣 = pygame.transform.smoothscale(紅色硬幣, (空間大小, 空間大小))
    黑色硬幣路徑 = pygame.image.load('4row_black.png')
    黑色硬幣路徑 = pygame.transform.smoothscale(黑色硬幣路徑, (空間大小, 空間大小))
    版子畫面 = pygame.image.load('4row_board.png')
    版子畫面 = pygame.transform.smoothscale(版子畫面, (空間大小, 空間大小))

    玩家獲勝畫面 = pygame.image.load('4row_humanwinner.png')
    電腦獲勝畫面 = pygame.image.load('4row_computerwinner.png')
    平手畫面 = pygame.image.load('4row_tie.png')
    贏家框 = 玩家獲勝畫面.get_rect()
    贏家框.center = (int(畫面寬 / 2), int(畫面高 / 2))

    箭頭畫面 = pygame.image.load('4row_arrow.png')
    箭頭方向 = 箭頭畫面.get_rect()
    箭頭方向.left = 紅色空間.right + 10
    箭頭方向.centery = 紅色空間.centery

    是初始遊戲 = True

    while True:
        進行遊戲(是初始遊戲)
        是初始遊戲 = False


def 進行遊戲(是初始遊戲):
    if 是初始遊戲:
        # Let the computer go first on the first game, so the 玩家
        # can see how the tokens are dragged from the token piles.
        輪到該方放棋 = 電腦
        表示求助 = True
    else:
        # Randomly choose who goes first.
        if random.randint(0, 1) == 0:
            輪到該方放棋 = 電腦
        else:
            輪到該方放棋 = 玩家
        表示求助 = False

    # Set up a blank 版子 data structure.
    主要版子 = 取得新版子()

    while True: # 主函式 game loop
        if 輪到該方放棋 == 玩家:
            # Human 玩家's 輪到該方放棋.
            玩家移動(主要版子, 表示求助)
            if 表示求助:
                # 輪到該方放棋 off help arrow after the first move
                表示求助 = False
            if 是贏家(主要版子, 紅棋):
                贏家畫面 = 玩家獲勝畫面
                break
            輪到該方放棋 = 電腦 # switch to other 玩家's 輪到該方放棋
        else:
            # Computer 玩家's 輪到該方放棋.
            圓柱 = 電腦移動(主要版子)
            電腦移動動作(主要版子, 圓柱)
            使之移動(主要版子, 黑棋, 圓柱)
            if 是贏家(主要版子, 黑棋):
                贏家畫面 = 電腦獲勝畫面
                break
            輪到該方放棋 = 玩家 # switch to other 玩家's 輪到該方放棋

        if 是版子全滿(主要版子):
            # A completely filled 版子 means it's a tie.
            贏家畫面 = 平手畫面
            break

    while True:
        # Keep looping until 玩家 clicks the mouse or quits.
        畫版子(主要版子)
        展示畫面.blit(贏家畫面, 贏家框)
        pygame.display.update()
        頻率時間.tick()
        for event in pygame.event.get(): # event handling loop
            if event.type == QUIT or (event.type == KEYUP and event.key == K_ESCAPE):
                pygame.quit()
                sys.exit()
            elif event.type == MOUSEBUTTONUP:
                return


def 使之移動(版子, 玩家, 圓柱):
    最低處 = 取得最低限制的空格(版子, 圓柱)
    if 最低處 != -1:
        版子[圓柱][最低處] = 玩家


def 畫版子(版子, 額外的硬幣=None):
    展示畫面.fill(背景顏色)

    # draw tokens
    空格框 = pygame.Rect(0, 0, 空間大小, 空間大小)
    for x in 範圍(邊界寬):
        for y in 範圍(邊界高):
            空格框.topleft = (X軸 + (x * 空間大小), Y軸 + (y * 空間大小))
            if 版子[x][y] == 紅棋:
                展示畫面.blit(紅色硬幣, 空格框)
            elif 版子[x][y] == 黑棋:
                展示畫面.blit(黑色硬幣路徑, 空格框)

    # draw the extra token
    if 額外的硬幣 != None:
        if 額外的硬幣['顏色'] == 紅棋:
            展示畫面.blit(紅色硬幣, (額外的硬幣['x'], 額外的硬幣['y'], 空間大小, 空間大小))
        elif 額外的硬幣['顏色'] == 黑棋:
            展示畫面.blit(黑色硬幣路徑, (額外的硬幣['x'], 額外的硬幣['y'], 空間大小, 空間大小))

    # draw 版子 over the tokens
    for x in 範圍(邊界寬):
        for y in 範圍(邊界高):
            空格框.topleft = (X軸 + (x * 空間大小), Y軸 + (y * 空間大小))
            展示畫面.blit(版子畫面, 空格框)

    # draw the red and black tokens off to the side
    展示畫面.blit(紅色硬幣, 紅色空間) # red on the left
    展示畫面.blit(黑色硬幣路徑, 黑色空間) # black on the right


def 取得新版子():
    版子 = []
    for x in 範圍(邊界寬):
        版子.append([空處] * 邊界高)
    return 版子


def 玩家移動(版子, 是初始移動):
    拖延的硬幣 = False
    硬幣X, 硬幣Y = None, None
    while True:
        for event in pygame.event.get(): # event handling loop
            if event.type == QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == MOUSEBUTTONDOWN and not 拖延的硬幣 and 紅色空間.collidepoint(event.pos):
                # start of dragging on red token pile.
                拖延的硬幣 = True
                硬幣X, 硬幣Y = event.pos
            elif event.type == MOUSEMOTION and 拖延的硬幣:
                # update the position of the red token being dragged
                硬幣X, 硬幣Y = event.pos
            elif event.type == MOUSEBUTTONUP and 拖延的硬幣:
                # let go of the token being dragged
                if 硬幣Y < Y軸 and 硬幣X > X軸 and 硬幣X < 畫面寬 - X軸:
                    # let go at the top of the screen.
                    圓柱 = int((硬幣X - X軸) / 空間大小)
                    if 是有效的移動(版子, 圓柱):
                        掉落硬幣的動作(版子, 圓柱, 紅棋)
                        版子[圓柱][取得最低限制的空格(版子, 圓柱)] = 紅棋
                        畫版子(版子)
                        pygame.display.update()
                        return
                硬幣X, 硬幣Y = None, None
                拖延的硬幣 = False
        if 硬幣X != None and 硬幣Y != None:
            畫版子(版子, {'x':硬幣X - int(空間大小 / 2), 'y':硬幣Y - int(空間大小 / 2), '顏色':紅棋})
        else:
            畫版子(版子)

        if 是初始移動:
            # Show the help arrow for the 玩家's first move.
            展示畫面.blit(箭頭畫面, 箭頭方向)

        pygame.display.update()
        頻率時間.tick()


def 掉落硬幣的動作(版子, 圓柱, 顏色):
    x = X軸 + 圓柱 * 空間大小
    y = Y軸 - 空間大小
    掉落速度 = 1.0

    最低空格 = 取得最低限制的空格(版子, 圓柱)

    while True:
        y += int(掉落速度)
        掉落速度 += 0.5
        if int((y - Y軸) / 空間大小) >= 最低空格:
            return
        畫版子(版子, {'x':x, 'y':y, '顏色':顏色})
        pygame.display.update()
        頻率時間.tick()


def 電腦移動動作(版子, 圓柱):
    x = 黑色空間.left
    y = 黑色空間.top
    速度 = 1.0
    # moving the black 鋪棋 up
    while y > (Y軸 - 空間大小):
        y -= int(速度)
        速度 += 0.5
        畫版子(版子, {'x':x, 'y':y, '顏色':黑棋})
        pygame.display.update()
        頻率時間.tick()
    # moving the black 鋪棋 over
    y = Y軸 - 空間大小
    速度 = 1.0
    while x > (X軸 + 圓柱 * 空間大小):
        x -= int(速度)
        速度 += 0.5
        畫版子(版子, {'x':x, 'y':y, '顏色':黑棋})
        pygame.display.update()
        頻率時間.tick()
    # dropping the black 鋪棋
    掉落硬幣的動作(版子, 圓柱, 黑棋)


def 電腦移動(版子):
    潛在移動 = 取得潛在移動(版子, 黑棋, 難度)
    # get the best fitness from the potential moves
    合格的最佳移動 = -1
    for i in 範圍(邊界寬):
        if 潛在移動[i] > 合格的最佳移動 and 是有效的移動(版子, i):
            合格的最佳移動 = 潛在移動[i]
    # find all potential moves that have this best fitness
    最佳移動 = []
    for i in 範圍(len(潛在移動)):
        if 潛在移動[i] == 合格的最佳移動 and 是有效的移動(版子, i):
            最佳移動.append(i)
    return random.choice(最佳移動)


def 取得潛在移動(版子, 鋪棋, 向前看):
    if 向前看 == 0 or 是版子全滿(版子):
        return [0] * 邊界寬

    if 鋪棋 == 紅棋:
        對手鋪棋 = 黑棋
    else:
        對手鋪棋 = 紅棋

    # Figure out the best move to make.
    潛在移動 = [0] * 邊界寬
    for 第一步移動 in 範圍(邊界寬):
        欺騙的版子 = copy.deepcopy(版子)
        if not 是有效的移動(欺騙的版子, 第一步移動):
            continue
        使之移動(欺騙的版子, 鋪棋, 第一步移動)
        if 是贏家(欺騙的版子, 鋪棋):
            # a winning move automatically gets a perfect fitness
            潛在移動[第一步移動] = 1
            break # don't bother calculating other moves
        else:
            # do other 玩家's counter moves and determine best one
            if 是版子全滿(欺騙的版子):
                潛在移動[第一步移動] = 0
            else:
                for 對抗手段 in 範圍(邊界寬):
                    欺騙的版子二 = copy.deepcopy(欺騙的版子)
                    if not 是有效的移動(欺騙的版子二, 對抗手段):
                        continue
                    使之移動(欺騙的版子二, 對手鋪棋, 對抗手段)
                    if 是贏家(欺騙的版子二, 對手鋪棋):
                        # a losing move automatically gets the worst fitness
                        潛在移動[第一步移動] = -1
                        break
                    else:
                        # do the recursive call to 取得潛在移動()
                        結論 = 取得潛在移動(欺騙的版子二, 鋪棋, 向前看 - 1)
                        潛在移動[第一步移動] += (sum(結論) / 邊界寬) / 邊界寬
    return 潛在移動


def 取得最低限制的空格(版子, 圓柱):
    # Return the row number of the 最低處 empty row in the given 圓柱.
    for y in 範圍(邊界高-1, -1, -1):
        if 版子[圓柱][y] == 空處:
            return y
    return -1


def 是有效的移動(版子, 圓柱):
    # Returns True if there is an empty space in the given 圓柱.
    # Otherwise returns False.
    if 圓柱 < 0 or 圓柱 >= (邊界寬) or 版子[圓柱][0] != 空處:
        return False
    return True


def 是版子全滿(版子):
    # Returns True if there are no empty spaces anywhere on the 版子.
    for x in 範圍(邊界寬):
        for y in 範圍(邊界高):
            if 版子[x][y] == 空處:
                return False
    return True


def 是贏家(版子, 鋪棋):
    # check horizontal spaces
    for x in 範圍(邊界寬 - 3):
        for y in 範圍(邊界高):
            if 版子[x][y] == 鋪棋 and 版子[x+1][y] == 鋪棋 and 版子[x+2][y] == 鋪棋 and 版子[x+3][y] == 鋪棋:
                return True
    # check vertical spaces
    for x in 範圍(邊界寬):
        for y in 範圍(邊界高 - 3):
            if 版子[x][y] == 鋪棋 and 版子[x][y+1] == 鋪棋 and 版子[x][y+2] == 鋪棋 and 版子[x][y+3] == 鋪棋:
                return True
    # check / diagonal spaces
    for x in 範圍(邊界寬 - 3):
        for y in 範圍(3, 邊界高):
            if 版子[x][y] == 鋪棋 and 版子[x+1][y-1] == 鋪棋 and 版子[x+2][y-2] == 鋪棋 and 版子[x+3][y-3] == 鋪棋:
                return True
    # check \ diagonal spaces
    for x in 範圍(邊界寬 - 3):
        for y in 範圍(邊界高 - 3):
            if 版子[x][y] == 鋪棋 and 版子[x+1][y+1] == 鋪棋 and 版子[x+2][y+2] == 鋪棋 and 版子[x+3][y+3] == 鋪棋:
                return True
    return False


if __name__ == '__main__':
    主函式()

