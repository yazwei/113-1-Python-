import random
import time
import os

class Character:
    def __init__(self, 名字):
        self.名字 = 名字
        self.生命值 = 100
        self.攻擊力 = 10
        self.裝備 = "木劍"
        self.補血瓶 = 2
        self.負面效果 = []

    def 攻擊(self, 敵人):
        骰子 = random.randint(1, 20)
        if "虛弱" in self.負面效果 and random.random() < 0.2:
            CLR()
            print(f"{self.名字} 處於虛弱狀態，攻擊失效！")
            return

        傷害 = self.攻擊力 + random.randint(1, 10)  # 基於攻擊力計算傷害
        if "虛弱" in self.負面效果:
            傷害 -= 5

        if 骰子 > 10:
            敵人.生命值 -= 傷害
            CLR()
            print(f"{self.名字} 的攻擊命中！對 {敵人.名字} 造成了 {傷害} 點傷害！")
        else:
            CLR()
            print(f"{self.名字} 的攻擊未命中！{敵人.名字} 反擊！")
            self.被攻擊()

    def 被攻擊(self):
        傷害 = random.randint(5, 15)
        self.生命值 -= 傷害
        CLR()
        print(f"{self.名字} 被攻擊，失去了 {傷害} 點生命值！")

    def 使用補血瓶(self):
        if self.補血瓶 > 0:
            self.補血瓶 -= 1
            self.生命值 += 25
            CLR()
            print(f"{self.名字} 使用了補血瓶，恢復了 25 點生命值！")
        else:
            CLR()
            print(f"{self.名字} 沒有補血瓶可用！")

    def 是否存活(self):
        return self.生命值 > 0

    def 更新狀態(self):
        if "中毒" in self.負面效果:
            減少血量 = random.randint(5, 8)
            self.生命值 -= 減少血量
            CLR()
            print(f"{self.名字} 因中毒失去了 {減少血量} 點生命值！")
        if self.負面效果:
            self.負面效果 = self.負面效果[1:]  # 每回合移除一個負面效果

class 探險:
    @staticmethod
    def 觸發事件(玩家, 機率):
        骰子 = random.randint(1, 100)
        if 骰子 <= 機率["敵人"]:
            探險.遇到敵人(玩家, "哥布林")
        elif 骰子 <= 機率["寶箱"]:
            探險.發現寶箱(玩家)
        elif 骰子 <= 機率["陷阱"]:
            探險.觸發陷阱(玩家)
        else:
            CLR()
            print("什麼也沒有發生。")

    @staticmethod
    def 遇到敵人(玩家, 敵人名字):
        敵人 = Character(敵人名字)
        敵人.生命值 = 30  # 哥布林的初始生命值
        CLR()
        print(f"你遇到了 {敵人名字}！")
        探險.戰鬥(玩家, 敵人)

    @staticmethod
    def 發現寶箱(玩家):
        增加攻擊 = random.randint(10, 15)
        玩家.攻擊力 += 增加攻擊
        CLR()
        print(f"你打開了寶箱，獲得了一顆力量寶石，攻擊力提升了 {增加攻擊} 點！")

    @staticmethod
    def 觸發陷阱(玩家):
        狀態 = random.choice(["中毒", "虛弱"])
        玩家.負面效果.append(狀態)
        CLR()
        print(f"你踩中了陷阱，現在處於 {狀態} 狀態！")

    @staticmethod
    def 戰鬥(玩家, 敵人):
        while 玩家.是否存活() and 敵人.是否存活():
            CLR()
            print(f"玩家狀態：生命值 {玩家.生命值}, 裝備 {玩家.裝備}, 補血瓶 {玩家.補血瓶}")
            print(f"敵人狀態：{敵人.名字}, 生命值 {敵人.生命值}")
            行動 = input("選擇行動：1. 攻擊  2. 使用補血瓶: ")
            if 行動 == "1":
                玩家.攻擊(敵人)
            elif 行動 == "2":
                玩家.使用補血瓶()
            else:
                CLR()
                print("無效選擇，失去一次行動！")

            if 敵人.是否存活():
                敵人.攻擊(玩家)

        CLR()
        if 玩家.是否存活():
            print(f"戰鬥結束！你擊敗了 {敵人.名字}！")
        else:
            print(f"你被 {敵人.名字} 打敗了！")

import os
import time

def CLR():
    for i in range(1, 4):
        # 使用 \r 清除當前行並顯示新內容
        print('\r' + '.' * i , end='', flush=True)
        time.sleep(0.5)
    print()  # 清屏動畫結束後換行
    
    # 清除畫面
    os.system('cls' if os.name == 'nt' else 'clear')

def 第一階段(玩家):
    while True:
        if input("是否進入洞窟? 如果你選擇不進入，可以洞窟周圍觀察一下\n你的答覆是?(是/否)") != "是":
            CLR()
            print("你在洞窟周圍觀察許久後什麼都發生，你或許該試著進入洞窟探索")
        else:
            break
    CLR()
    print("第 1 階段探索開始！")
    time.sleep(1)
    玩家.更新狀態()
    探險.觸發事件(玩家, {"敵人": 40, "寶箱": 50, "陷阱": 60})

def 第二階段(玩家):
    if input("是否進行第 2 階段探索？(是/否): ") != "是":
        CLR()
        print("你選擇不探索，冒險結束。")
        return False

    CLR()
    print("第 2 階段探索開始！")
    time.sleep(1)
    玩家.更新狀態()
    探險.觸發事件(玩家, {"敵人": 40, "寶箱": 60, "陷阱": 70})

def 第三階段(玩家):
    if input("是否進行第 3 階段探索？(是/否): ") != "是":
        CLR()
        print("你選擇不探索，冒險結束。")
        return False

    CLR()
    print("第 3 階段探索開始！")
    time.sleep(1)
    玩家.更新狀態()
    探險.觸發事件(玩家, {"敵人": 30, "寶箱": 60, "陷阱": 70})

def 第四階段(玩家):
    if input("是否進行第 4 階段探索？(是/否): ") != "是":
        CLR()
        print("你選擇不探索，冒險結束。")
        return False

    CLR()
    print("第 4 階段探索開始！")
    time.sleep(1)
    print("你遇到了哥布林領主！")
    領主 = Character("哥布林領主")
    領主.生命值 = 100                                           #哥布林領主血量

    while 玩家.是否存活():
        CLR()
        print(f"玩家狀態：生命值 {玩家.生命值}, 裝備 {玩家.裝備}, 補血瓶 {玩家.補血瓶}")
        print(f"哥布林領主：生命值 {領主.生命值}")
        行動 = input("選擇行動：1. 攻擊哥布林領主  2. 使用補血瓶  3. 逃跑: ")
        if 行動 == "1":
            玩家.攻擊(領主)
            if 領主.是否存活():
                領主.攻擊(玩家)
        elif 行動 == "2":
            玩家.使用補血瓶()
        elif 行動 == "3":
            CLR()
            print(f"勇者{玩家.名字}選擇了逃跑公主被永遠的困在了洞窟中。")
            print("遊戲結束")
            return
        else:
            CLR()
            print("無效選擇，失去一次行動！")

        if not 領主.是否存活():
            CLR()
            print(f"恭喜！{玩家.名字} 打敗了哥布林領主，完成了冒險！")
            print(f"從此之後{玩家.名字} 與公主過著幸福快樂的日子。")
            return

    if not 玩家.是否存活():
        CLR()
        print(f"很遺憾！{玩家.名字} 被哥布林領主打敗了！")

def 主程序():
    print("歡迎來到簡易D&D遊戲！")

    print("背景設定:")
    print("在阿拉希亞王國的邊境，坐落著一個名為\"陰影山谷\"的險地。傳聞這裡曾是古代精靈的庇護所，但如今卻成了哥布林的巢穴。一週前，公主，被一群突襲的哥布林擄走，帶入了山谷深處的洞窟中。傳言洞窟內盤踞著一位狡詐且擁有強大的哥布林首領，它用古老的詛咒保護著洞穴。")

    名字 = input("玩家將作為一位受國王委託前往拯救公主的勇士，請輸入玩家名稱後遊戲開始: ")
    玩家 = Character(名字)
    CLR()
    print(f"{玩家.名字}經歷了長途跋涉後終於抵達了公主綁架的洞窟")
    第一階段(玩家)
    if not 玩家.是否存活():
        CLR()
        print(f"很遺憾！{玩家.名字} 犧牲了！")
        return

    第二階段(玩家)
    if not 玩家.是否存活():
        CLR()
        print(f"很遺憾！{玩家.名字} 犧牲了！")
        return

    第三階段(玩家)
    if not 玩家.是否存活():
        CLR()
        print(f"很遺憾！{玩家.名字} 犧牲了！")
        return

    第四階段(玩家)

if __name__ == "__main__":
    主程序()
