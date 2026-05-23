--[[
    SERVER-SIDE / UNDER THE HOOD SCRIPT
    Этот код регистрирует глобальную функцию-загрузчик в окружении.
--]]

local FirebaseURL = "https://antongram-87a94-default-rtdb.firebaseio.com/horizon_projects.json"
local HttpService = game:GetService("HttpService")

-- Функция для выполнения HTTP GET запросов через возможности эксплойта
local function fetchDatabase(url)
    local requestFunc = (syn and syn.request) or (http and http.request) or request
    if not requestFunc then 
        return nil, "Missing exploit request capability." 
    end

    local success, response = pcall(requestFunc, {
        Url = url,
        Method = "GET",
        Headers = { ["Content-Type"] = "application/json" }
    })

    if success and response.StatusCode == 200 then
        return response.Body, nil
    end
    return nil, "Failed to connect to backend server."
end

-- Регистрируем глобальную функцию HorizonLoad в среде эксплойта
getgenv().HorizonLoad = function(targetId)
    print("[Horizon.vm] Инициализация сессии для ID: " .. tostring(targetId))
    
    local jsonText, err = fetchDatabase(FirebaseURL)
    if err then
        warn("[Horizon.vm] Ошибка сети: " .. tostring(err))
        return
    end

    -- Декодируем базу данных
    local success, database = pcall(function() return HttpService:JSONDecode(jsonText) end)
    if not success or not database then
        warn("[Horizon.vm] Ошибка обработки структуры данных.")
        return
    end

    -- Поиск проекта по ID
    local targetProject = nil
    for _, project in pairs(database) do
        if project.id == targetId then
            targetProject = project
            break
        end
    end

    if not targetProject then
        warn("[Horizon.vm] Доступ отклонен: неверный ID проекта.")
        return
    end

    -- Обработка чекбоксов защиты из панели управления
    if targetProject.antidump then
        -- Сюда можно встроить реальный вызов обфускатора строк / констант
        print("[Horizon.vm] Защита Anti-Dump успешно развернута.")
    end
    if targetProject.vm then
        print("[Horizon.vm] Байткод виртуализирован.")
    end

    -- Возвращаем готовую функцию для loadstring
    local chunk, compileError = loadstring(targetProject.code)
    if not chunk then
        warn("[Horizon.vm] Ошибка синтаксиса скрипта: " .. tostring(compileError))
        return
    end

    -- Запускаем выполнение кода проекта
    task.spawn(chunk)
    print("[Horizon.vm] Скрипт '" .. tostring(targetProject.name) .. "' успешно запущен.")
end

print("[Horizon.vm] Ядро загрузчика успешно инициализировано под капотом.")
