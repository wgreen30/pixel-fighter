import pygame


class Fighter():
    def __init__(self, x, y, Flip, data, sprite_sheet, animation_steps):
        self.player = data[5]
        self.size_x = data[0][0]
        self.size_y = data[0][1]
        self.offset = data[2]
        self.image_scale = data[1]
        self.dict = data[3]
        self.timing = data[4]
        self.animation_list = self.load_images(sprite_sheet, animation_steps)
        self.action = self.dict["IDLE"]
        self.frame_index = 0
        self.frame_time = 100
        self.image = self.animation_list[self.action][self.frame_index]
        self.mask = pygame.mask.from_surface(self.image)
        self.update_time = pygame.time.get_ticks()
        self.flip = Flip
        self.rect = pygame.Rect(x, y, 200, 250) #160, 250
        self.vel_y = 0
        self.vel_x = 0
        self.running = False
        self.jump = False
        self.attacking = False
        self.attack_type = 0
        self.attack_cooldown = 0
        self.hit = False
        self.health = 100
        self.alive = True
        if self.player == 1:
            self.facing_right = True
            self.facing_left = False
        if self.player == 2:
            self.facing_right = False
            self.facing_left = True
        self.dashing = False
        self.dash_status = 250



    def load_images(self, sprite_sheet, animation_steps):
        #extract images from spritesheet
        animation_list = []
        temp_count = 0
        for y, animation in enumerate(animation_steps):
            temp_img_list = []
            for x in range(animation):
                temp_img = sprite_sheet.subsurface(x * self.size_x, y * self.size_y, self.size_x, self.size_y)
                temp_img_scaled = pygame.transform.scale(temp_img, (self.image_scale * self.size_x, self.image_scale * self.size_y))
                temp_img_list.append(temp_img_scaled)
            animation_list.append(temp_img_list)
        return animation_list


        pass

    def move(self, screen_width, screen_height, ground_level, surface, target):
        SPEED = 15
        GRAVITY = 3
        DECC = 5
        dx = 0
        dy = 0
        self.running = False

        #get key presses
        key = pygame.key.get_pressed()
        #can only perform other actions if not currently attacking
        if self.alive:
            if not self.dashing:
                if not self.attacking:
                    # movement
                    if self.player == 1:
                        if key[pygame.K_a]:
                            dx = -SPEED
                            self.facing_left = True
                            self.facing_right = False
                            self.running = True
                        elif key[pygame.K_d]:
                            dx = SPEED
                            self.facing_left = False
                            self.facing_right = True
                            self.running = True
                        # jump
                        if key[pygame.K_w] and not self.jump:
                            self.vel_y = -45
                            self.jump = True
                        # Attack
                        if key[pygame.K_r] or key[pygame.K_t]:
                            self.attack(surface, target)
                            # dtermine which attack was used
                            if key[pygame.K_r]:
                                self.attack_type = 1
                            if key[pygame.K_t]:
                                self.attack_type = 2
                        # Dash
                        if key[pygame.K_LSHIFT] and self.dash_status >= 250:
                            self.dash_status = 0
                            self.dashing = True
                            if self.facing_left:
                                self.vel_x = -100
                            elif self.facing_right:
                                self.vel_x = 100
                    if self.player == 2:
                        if key[pygame.K_LEFT]:
                            dx = -SPEED
                            self.facing_left = True
                            self.facing_right = False
                            self.running = True
                        elif key[pygame.K_RIGHT]:
                            dx = SPEED
                            self.facing_left = False
                            self.facing_right = True
                            self.running = True
                        # jump
                        if key[pygame.K_UP] and not self.jump:
                            self.vel_y = -45
                            self.jump = True
                        # Attack
                        if key[pygame.K_m] or key[pygame.K_n]:
                            self.attack(surface, target)
                            # dtermine which attack was used
                            if key[pygame.K_m]:
                                self.attack_type = 1
                            if key[pygame.K_n]:
                                self.attack_type = 2
                        # Dash
                        if key[pygame.K_RSHIFT] and self.dash_status >= 250:
                            self.dash_status = 0
                            self.dashing = True
                            if self.facing_left:
                                self.vel_x = -100
                            elif self.facing_right:
                                self.vel_x = 100


        #DECC
        if self.dashing:
            if self.vel_x > 0:
                self.vel_x -= DECC
            if self.vel_x < 0:
                self.vel_x += DECC
            self.vel_y = 0

        if not self.dashing:
            # gravity
            self.vel_y += GRAVITY
            self.vel_x = 0
        dy += self.vel_y
        dx += self.vel_x

        #Keep on screen
        if self.rect.left + dx < 0:
            dx = -self.rect.x
        if self.rect.right + dx > screen_width:
            dx = screen_width - self.rect.right
        if (self.rect.bottom) + dy > screen_height - ground_level:
            self.vel_y = 0
            self.jump = False
            dy = screen_height - ground_level - self.rect.bottom

        #insure plkayers face eachother
        if target.rect.centerx > self.rect.centerx:
            self.flip = False
        else:
            self.flip = True

    #apply attack cooldown
        if self.attack_cooldown > 0:
            self.attack_cooldown -= 1
        if self.dash_status < 250:
            self.dash_status += 3

        #update player position
        self.rect.x += dx
        self.rect.y += dy
    #handle animation updates
    def update(self):
        #check what action the player is performing
        if self.health <= 0:
            self.health = 0
            self.alive = False
            self.update_action(self.dict["DEATH"])
        elif self.dashing:
            self.update_action(self.dict["DASH"])
        elif self.hit:
            self.update_action(self.dict["HURT"])
        elif self.attacking == True:
            if self.attack_type == 1:
                self.update_action(self.dict["ATTACK 1"])
            elif self.attack_type == 2:
                self.update_action(self.dict["ATTACK 2"])
        elif self.jump:
            self.update_action(self.dict["JUMP"])
        elif self.running:
            self.update_action(self.dict["RUN"])
        else:
            self.update_action(self.dict["IDLE"])

        animation_cooldown = 1000
        self.image = self.animation_list[self.action][self.frame_index]
        self.mask = pygame.mask.from_surface(self.image)
        self.frame_timing()


    def attack(self, surface, target):
        attack1 = 130
        attack2 = 50
        self.attacking = True

        if self.attacking:
            attack_rect = pygame.Rect(0, 0, 0, 0)
            if self.facing_right and self.attack_type == 1:
                attack_rect = pygame.Rect(self.rect.right, self.rect.y, attack1, self.rect.height)
            if self.facing_left and self.attack_type == 1:
                attack_rect = pygame.Rect(self.rect.x - attack1 , self.rect.y, attack1, self.rect.height)

        if attack_rect.colliderect(target.rect):
                target.health -= 10
                print("hit")

    def update_action(self, new_action):
        #check if the new action is different than the previous one
        if new_action != self.action:
            self.action = new_action
            #update the animation settings
            self.frame_index = 0
            self.update_time = pygame.time.get_ticks()
    def frame_timing(self):
        #idle handling 8, 11
        if self.action == self.dict["IDLE"]:
            self.frame_time = self.timing["IDLE"]
        #run handling 10, 15
        if self.action == self.dict["RUN"]:
            self.frame_time = self.timing["RUN"]
        #jump handling
        if self.action == self.dict["JUMP"]:
            self.frame_time = self.timing["JUMP"]
        #attack handling
        if self.action == self.dict["ATTACK 1"]:
            self.frame_time = self.timing["ATTACK 1"]
        if self.action == self.dict["ATTACK 2"]:
            self.frame_time = self.timing["ATTACK 2"]
        #dash handling
        if self.action == self.dict["DASH"]:
            self.frame_time = self.timing["DASH"]

        if pygame.time.get_ticks() - self.update_time > self.frame_time:
            self.frame_index += 1
            self.update_time = pygame.time.get_ticks()
        #check if the animation has finsihed
            if self.frame_index >= len(self.animation_list[self.action]):
                #if the player is dead then end the animation
                if self.alive == False:
                    self.frame_index = len(self.animation_list[self.action]) - 1
                else:
                    self.frame_index = 0
                #check if an attack was executed
                if (self.action == self.dict["ATTACK 1"] or self.action == self.dict["ATTACK 2"]):
                    self.attacking = False
                    self.attack_cooldown = 0
                if self.action == self.dict["HURT"]:
                    self.hit = False
                    #if the plater was in the middel of an attack, then the attack is stopped
                    self.attacking = False
                    self.attack_cooldown = 0
                if self.action == self.dict["JUMP"]:
                    self.frame_index = len(self.animation_list[self.action]) - 1
                if self.dashing:
                    self.dashing = False
    def draw(self, surface):
        img = pygame.transform.flip(self.image, True, False)
        if self.facing_right:
            surface.blit(self.image, (self.rect.x - self.offset[0], self.rect.y - self.offset[1]))
        elif self.facing_left:
            surface.blit(img, (self.rect.x - self.offset[0], self.rect.y - self.offset[1]))
