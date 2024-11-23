road_width, height))
    
    # Add lane markers
    for y in range(0, height, lane_marker_height * 2):
        for lane in lanes:
            pygame.draw.rect(screen, white, (lane + 45 - 10, y, 10, lane_marker_height))

    # Draw player and vehicles
    player_group.draw(screen)
    vehicle_group.draw(screen)

    # Vehicle movement and collision check
    for vehicle in vehicle_group:
        vehicle.rect.y += speed
        if vehicle.rect.top > height:
            vehicle.kill()
            score += 1
            if score % 5 == 0:  # Increase speed every 5 points
                speed += 1
    
    if len(vehicle_group) < 2:
        add_vehicle = True
        for vehicle in vehicle_group:
            if vehicle.rect.top < vehicle.rect.height * 1.5:
                add_vehicle = False
        if add_vehicle:
            lane = random.choice(lanes)
            image = random.choice(vehicle_images)
            vehicle_group.add(Vehicle(image, lane, -height // 2))
    
    # Display score
    display_score()
    
    # Check for collisions
    if pygame.sprite.spritecollide(player, vehicle_group, True):
        gameover = True
        crash_rect.center = player.rect.center

    if gameover:
        game_over_screen()
        while gameover:
            for event in pygame.event.get():
                if event.type == QUIT:
                    gameover = False
                    running = False
                if event.type == KEYDOWN:
                    if event.key == K_y:
                        reset_game()
                    elif event.key == K_n:
                        running = False
                        gameover = False
    
    pygame.display.update()

